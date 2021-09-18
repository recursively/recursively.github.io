---
title: Sonar Code Quality Gate Integration with CI Part 3
date: 2021-09-18 09:50:54
categories: SAST
tags: [sonarqube, sonar-scanner, Jenkins, Gitlab]
keywords: [sonarqube, sonar-scanner, Jenkins, Gitlab]
description: This post is to solve the potential problems of sonar scanner TLS verification, if the certificate in use is a self-signed certificate.
---
## Certificate Generation

The default sonarqube service runs as HTTP service, we're going to generate our self-signed certificate. Since I've been deploying the sonarqube service through kubernetes with nginx as ingress, I just use self-signed certificate for convenience. Besides this, you can also try out the [Let's Encrypt](https://letsencrypt.org/) tool to generate the browser-recognized certificate, the nginx ingress supports this way well.

The *san.cnf* configuration file is necessary for generating the certificate which will be used by the sonar scanner SSL verification procedure. If you want your certificates to support Subject Alternative Names (SANs), you must define the alternative names in a configuration file.

```ini
[req]
default_bits  = 2048
distinguished_name = req_distinguished_name
req_extensions = req_ext
x509_extensions = v3_req
prompt = no

[req_distinguished_name]
countryName = XX
stateOrProvinceName = N/A
localityName = N/A
organizationName = XX
commonName = www.example.com

[req_ext]
subjectAltName = @alt_names
[v3_req]
subjectAltName = @alt_names
[alt_names]
DNS.1 = www.example.com
IP.1 = x.x.x.x
```

```shell
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 730 -out cert.pem -config san.cnf
```

## Truststore Generation

```shell
keytool -trustcacerts -keystore truststore.jks -alias abc -import -file cert.pem
```

Set the sonar environment variables to invoke the truststore file.

```shell
export SONAR_SCANNER_OPTS="-Djavax.net.ssl.trustStore=/path/truststore.jks -Djavax.net.ssl.trustStorePassword=changeit"
```

The sonar scanner will work normally then.

## References

http://doc.isilon.com/ECS/3.2/AdminGuide/ecs_t_certificate_generate_with_san.html

https://stackoverflow.com/questions/47434877/how-to-generate-keystore-and-truststore

https://sionwilliams.com/posts/2019-04-25_sonar_scanner_certs/