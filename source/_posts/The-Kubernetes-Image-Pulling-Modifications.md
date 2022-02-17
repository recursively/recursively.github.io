---
title: The Kubernetes Image Pulling Modifications
date: 2021-08-20 17:57:38
categories: Cloud
tags: [kubernetes, k8s]
keywords: [kubernetes, k8s]
description: It's hard to pull some kind of kubernetes images by some reasons. So I just wrote down some approaches to deal with that tricky problem, which may save a lot of time.
---
## Set up the CRI Proxy

Set up the proxy for the CRI(Container Runtime Interface) will solve the connection problem in most situations. Considering most of people use docker to pull the container images and docker has containerd which is also a CRI, we can create configuration files and add some parameters.

Create the docker configuration file:

```shell
mkdir -p /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/http-proxy.conf
```
Modify the *http-proxy.conf* file:
```conf
[Service]
Environment="HTTP_PROXY=http://x.x.x.x"
Environment="HTTPS_PROXY=http://x.x.x.x"
Environment="NO_PROXY=172.20.1.150"
```

Create the containerd configuration file:

```shell
mkdir -p /etc/systemd/system/containerd.service.d
touch /etc/systemd/system/containerd.service.d/http-proxy.conf
```
Modify the *http-proxy.conf* file:
```conf
[Service]
Environment="HTTP_PROXY=http://x.x.x.x"
Environment="HTTPS_PROXY=http://x.x.x.x"
Environment="NO_PROXY=172.20.1.150"
```

## Set up Private Registry

For security concerns, it will be better to create a private registry with authentication. Here I'm going to use the basic authentication to achieve that.

Create a password file with one entry for the user testuser, with password testpassword:

```shell
mkdir auth
docker run --rm \
  --entrypoint htpasswd \
  httpd:2 -Bbn testuser testpassword > auth/htpasswd
```
We're going to use the TLS channel to access the registry server, so we have to generate the certificate ourselves. Otherwise you can use the *insecure-registries* parameter to bypass the HTTPS warning. Edit the *daemon.json* file, whose default location is /etc/docker/daemon.json on Linux:

```json
{
  "insecure-registries" : ["myregistrydomain.com:5000"]
}
```
Let's create the certificate and key file:

```
mkdir certs && cd ./certs
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem -subj "/CN=example.com" -addext "subjectAltName = DNS:example.com"
```
Install the certificate for your local machine, but it seems that docker will not refer to the Linux certificates when it connects to the registry.

```shell
cp certs/certificate.pem /usr/local/share/ca-certificates/registry-certificate.crt
sudo update-ca-certificates
```
Now we've install the self-signed certificate properly. Note that if you want to enable the deleting switch which allows you to delete the images on the registry, you need to create a yaml file and edit it like this: 

```yaml
version: 0.1
log:
  fields:
    service: registry
storage:
  delete:
    enabled: true
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: localhost:5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

Let's start the registry with basic authentication:

```shell
docker run -d \
  --name registry \
  -v "$(pwd)"/config.yml:/etc/docker/registry/config.yml \
  -v /data/registry:/var/lib/registry \
  -v "$(pwd)"/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -v "$(pwd)"/certs:/certs \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/certificate.pem \
  -e REGISTRY_HTTP_TLS_KEY=/certs/key.pem \
  -p 6000:443 \
  registry
```

Connection test:

```shell
curl -XGET https://localhost:6000/v2/_catalog -u reporter:58fd24d4311a742d13464373398ff3d9
```

Maybe you have to modify the image pulling credentials in your kubernetes yaml file.

Create a secret named regcred:

```shell
kubectl create secret docker-registry regcred \
  --docker-server=<your-registry-server> \
  --docker-username=<your-name> \
  --docker-password=<your-pword> \
  --docker-email=<your-email>
```

And add the credential in your pod:

```shell
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:
  - name: regcred
```

If you are using a virtual machine manager for kubernetes like kubevirt, it will be a little different:

```shell
 volumes:
   - name: containervolume
     containerDisk:
       image: <your-private-image>
       imagePullSecret: regcred
```

### Push Images

Since we have enabled the basic authentication, you have to execute the login command first and foremost.

```shell
docker login http://172.20.1.150:6000
```
After entering your username and password, the *~/.docker/config.json* file will be generated automatically. 

Then you can push your images to the registry by a few commands:

```shell
docker image tag rhel-httpd:latest registry-host:6000/myadmin/rhel-httpd:latest

docker image push registry-host:6000/myadmin/rhel-httpd:latest
```

### Image Deleting

Commonly we get all the images kept on the registry by using the v2 api:
```shell
curl -XGET https://localhost:6000/v2/_catalog -u name:password --insecure
curl -XGET https://localhost:6000/v2/<name>/tags/list -u name:password --insecure
```
We can also list all the images along with their tags with a python script:
```python
#-*- coding:utf-8 -*-

import requests
import json

repo_ip = '172.20.1.150'
repo_port = 6000
auth_name = 'name'
auth_password = 'password'

def getImagesNames(repo_ip,repo_port):
    docker_images = []
    try:
        url = "https://" + repo_ip + ":" +str(repo_port) + "/v2/_catalog"
        res =requests.get(url, auth=(auth_name, auth_password)).content.strip()
        res_dic = json.loads(res)
        images_type = res_dic['repositories']
        for i in images_type:
            url2 = "https://" + repo_ip + ":" +str(repo_port) +"/v2/" + str(i) + "/tags/list"
            res2 = requests.get(url2, auth=(auth_name, auth_password)).content.strip()
            res_dic2 = json.loads(res2)
            name = res_dic2['name']
            tags = res_dic2['tags']
            if tags:
                for tag in tags:
                    docker_name = str(repo_ip) + ":" + str(repo_port) + "/" + name + ":" + tag
                    docker_images.append(docker_name)
                    print(docker_name)
            else:
                pass
    except Exception as e:
        print(e)

    return docker_images

getImagesNames(repo_ip, repo_port)
```
Run this script:
```shell
python3 get_image.py
```

We can use the v2 api to delete a specific image:
```shell
curl -XDELETE https://localhost:6000/v2/<name>/manifests/<reference> -u name:password
```

For example:

```shell
curl -XDELETE https://localhost:6000/v2/<name>/manifests/sha256:60f2883fbfca71ff7740a6eca7bd8bd466988031dcf55093bb8ff2b26f2c5479 -u name:password --insecure
```

The name is the image name you want to delete, and the reference is the digest of the targeted image. To get the digest of a image with docker:
```shell
docker image ls --digests image_name
```

Or retrieve the digest from the registry server with command:

```shell
curl -X GET --header "Accept: application/vnd.docker.distribution.manifest.v2+json" -I 
http://x.x.x.x:5000/v2/
```

For example:

```shell
curl -XGET --header "Accept: application/vnd.docker.distribution.manifest.v2+json" -I https://localhost:6000/v2/<name>/manifests/latest -u name:password --insecure
```

The response will be like this:

```shell
HTTP/2 200
content-type: application/vnd.docker.distribution.manifest.v2+json
docker-content-digest: sha256:60f2883fbfca71ff7740a6eca7bd8bd466988031dcf55093bb8ff2b26f2c5479
docker-distribution-api-version: registry/2.0
etag: "sha256:60f2883fbfca71ff7740a6eca7bd8bd466988031dcf55093bb8ff2b26f2c5479"
x-content-type-options: nosniff
content-length: 1582
date: Sat, 18 Sep 2021 03:37:20 GMT
```

Finally, to make the deleting action take effect, you need to get into the container and execute the garbage-collection command:

```shell
registry garbage-collect /etc/docker/registry/config.yml
```
Then you can check the storage to verify:
```shell
du -chs /var/lib/registry/
```

## Using the Cloud Service Provider's Registry



## References

https://docs.docker.com/registry/insecure/

https://docs.docker.com/registry/deploying/

https://docs.docker.com/registry/spec/api/