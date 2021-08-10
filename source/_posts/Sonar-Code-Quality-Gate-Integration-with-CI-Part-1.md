---
title: Sonar Code Quality Gate Integration with CI - Part 1
date: 2021-07-28 17:19:02
categories: SAST
tags: [sonarqube, sonar-scanner, Jenkins, Gitlab]
keywords: [sonarqube, sonar-scanner, Jenkins, Gitlab]
description: Basically, there are a couple of approaches to handle the code quality checking in different stage, such as pre-commit, pre-merge and post-merge. This post is to state the code quality checking process and the integration with CI in the pre-merge stage, which is usually known as quality gate.
---
## Why Sonar
There are a lot of SAST tools on the market, such as Fortify, Coverity, Infer and so on. We choose Sonar because of its open-source(partially), convenient-to-deployed, customization-friendly properties.

## About CI
For the CI system, we're gonna talk about two platforms:
* Jenkins CI
* Gitlab CI

The Jenkis CI is a third-party CI system and Gitlab CI is originated from Gitlab on the contrary. The deployment experience on both platforms can be applied to other situations.

## Trigger A Gitlab Pipeline Separatly
There are three steps to accomplish this: 
* Set up Gitlab runner
* Register a runner for your project
* Create the *.gitlab-ci.yml* file in your project

Go to Settings > CI/CD and expand Runners to see if you have available runners. You could notice that there's a token listed on the board, it will be used for the gitlab-runner configuration later. If no runners are listed on the page you must install Gitlab runner:
```shell
sudo apt install gitlab-runner
```
After that, you need to register a runner to perform your task:
```shell
sudo gitlab-runner register
```
You need to enter some information the runner requires to finish the registration:
* Enter your GitLab instance URL.
* Enter the token you obtained to register the runner.
* Enter a description for the runner. You can change this value later in the GitLab user interface.
* Enter the tags associated with the runner, separated by commas. You can change this value later in the GitLab user interface. Please note that if you don't want your runner's tags associated with the ones you defined in Gitlab, you can modify the configuration in the runner's setting panel.

<img src="pic_1.png" width="100%" height="100%">

<img src="pic_2.png" width="100%" height="100%">

* Provide the runner executor. For most use cases, enter docker.
* If you entered docker as your executor, you’ll be asked for the default image to be used for projects that do not define one in .gitlab-ci.yml. If you don't want the docker as the executor to pull images every time, you need to add the *pull_policy* parameter in the configuration file: */etc/gitlab-runner/config.toml*
```toml
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "CI test project"
  url = "http://172.20.1.135"
  token = "Your project token"
  executor = "docker"
  [runners.custom_build_dir]
  [runners.cache]
    [runners.cache.s3]
    [runners.cache.gcs]
    [runners.cache.azure]
  [runners.docker]
    tls_verify = false
    image = "ubuntu:latest"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
    pull_policy = "if-not-present"
```

The last step is to create the *.gitlab-ci.yml* file which is used to trigger your Gitlab pipeline. In this file, you define:
* The structure and order of jobs that the runner should execute.
* The decisions the runner should make when specific conditions are encountered.

Now that most of the configuration is set up. Let's have a try to trigger the Gitlab CI job by pushing our code to the remote repository.

Create two simple c files which has these vulnerable code:
```C
int main()
{
    int x;
    x=121/0;
    return 0;
}
```

```C
int main()
{
    int x;
    int y=10;
    int a[10];
    x=a[y];
    return 0;

}
```
```makefile
# Makefile
all: test1 test2

test1:
        gcc -o vuln.o vuln.c

test2:
        gcc -o vuln2.o vuln2.c

clean:
        rm vuln*.o
```

Then add these content to the *.gitlab-ci.yml* file:
```yml
build-job:
  stage: build
  script:
    - apt update
    - apt install gcc make -y
    - echo "Starting up the building process!"
    - make
```
You can definitely build your own images without installing packages every time, which will save a lot of building time. Now you can commit and push your project to the remote repository to trigger the CI pipeline job. You can get the build status after a while.

<img src="pic_3.png" width="100%" height="100%">

And you can also check the build details if you will:

<img src="pic_4.png" width="100%" height="100%">

## Trigger A Jenkins Pipeline Separatly

The Jenkins pipeline task is mentioned in another post: 

## Trigger A Sonar Scanning Task Separatly

Now it's time to use Sonarqube to perform the code quality scanning. To accomplish that, we need to set up both Sonarqube and Sonar-scanner environment. First and foremost, create the Sonarqube server environment, it's easily to do that with a docker image:

```shell
docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest
```
When it's finished, you can access the Sonarqube web UI server through the browser. Let's create a scanning project and fill in these blank area. 

<img src="pic_5.png" width="100%" height="100%">

<img src="pic_6.png" width="100%" height="100%">

<img src="pic_7.png" width="100%" height="100%">

Since the community version of Sonarqube doesn't support the scanning for C/C++ code, we have to install plugins to enable the C/C++ checking rule. Sonar-cxx(https://github.com/SonarOpenCommunity/sonar-cxx) is a nice tool which has many sensors for reading reports generated by other scanning tools such as cppcheck and infer.

Here we use cppcheck to find the flaws in our code:
```shell
sudo apt install cppcheck

cppcheck --xml --xml-version=2 --enable=all ./ 2> cppcheck-report.xml
```

Create a configuration file in your project's root directory called *sonar-project.properties*, and add some configuration for cxx plugin:

```ini
# must be unique in a given SonarQube instance
sonar.projectKey=test

# --- optional properties ---

# defaults to project key
#sonar.projectName=My project
# defaults to 'not provided'
#sonar.projectVersion=1.0
 
# Path is relative to the sonar-project.properties file. Defaults to .
#sonar.sources=.
 
# Encoding of the source code. Default is default system encoding
#sonar.sourceEncoding=UTF-8

# mandatory: files to be handled by the _cxx plugin_
sonar.cxx.file.suffixes=.h,.cpp,.c

sonar.cxx.cppcheck.reportPaths=cppcheck-report.xml
```

Now we can start a Sonar-scanner job to perform our code scanning.
```shell
sudo docker run \
    --rm \
    -e SONAR_HOST_URL="http://172.20.1.135:9000" \
    -e SONAR_LOGIN="50b94782744687df5d5b04863b6a3c2198b3361a" \
    -v "/home/CI/awesome/:/usr/src" \
    sonarsource/sonar-scanner-cli
```
If everything goes well, you can find that all the findings are listed on the Sonarqube project board.

<img src="pic_8.png" width="100%" height="100%">

<img src="pic_9.png" width="100%" height="100%">

## Build Job & Sonar Scanner Combination

To perform both cppcheck and Sonarqube scanning, you need to modify your container to include both of the tools.
```shell
sudo docker run -it --rm CONTAINER_ID /bin/bash
```

Install the cppcheck package in the container:
```shell
bash-5.1# apk add cppcheck
```

Commit your modified container from another terminal in order to perform the CI scanning task later:
```shell
sudo docker commit CONTAINER_ID modified/sonar-scanner-cli
```

Create the *.gitlab-ci.yml* and edit the configuration:

```yml
build-job:
  stage: build
  script:
    - apt update
    - apt install gcc make -y
    - echo "Starting up the building process!"
    - make

sonarqube-check:
  image:
    name: modified/sonar-scanner-cli:latest
    entrypoint: [""]
  variables:
    SONAR_USER_HOME: "/home/CI/awesome/.sonar"  # Defines the location of the analysis task cache
    GIT_DEPTH: "0"  # Tells git to fetch all the branches of the project, required by the analysis task
  cache:
    key: "sonar ci"
    paths:
      - .sonar/cache
  script:
    - cppcheck --xml --xml-version=2 --enable=all ./ 2> cppcheck-report.xml
    - sonar-scanner
      -Dsonar.qualitygate.wait=true
      -Dsonar.host.url="http://172.20.1.135:9000"
      -Dsonar.login="50b94782744687df5d5b04863b6a3c2198b3361a"

  allow_failure: false
  only:
    - merge_requests
    - master
    - develop
```
Trigger the Gitlab CI pipeline and take a look at the status:

<img src="pic_10.png" width="100%" height="100%">

Furthermore, Sonarqube allows you to configure the quality gate conditions to decide what sort of code quality can pass the CI pipeline. 

In this scenario, we're gonna create the quality gate before the merge process, the merge request won't succeed if the the status of the quality gate is failed. First of all, we need to choose the checkbox below to enable the merge check.

<img src="pic_15.png" width="100%" height="100%">

Create a new quelity gate and add a reliability rating condition which decides whether passing the merge or not.

<img src="pic_11.png" width="100%" height="100%">

Now create a new branch and try merging this new branch into master branch, the code owner will get a notification like this:

<img src="pic_12.png" width="100%" height="100%">

If the code scanning failed, the merge button will be in red. And you can find that the pipeline failed because it did not pass the quality gate.

<img src="pic_13.png" width="100%" height="100%">

<img src="pic_14.png" width="100%" height="100%">

## References

https://docs.gitlab.com/runner/register/

https://docs.gitlab.com/runner/executors/docker.html

https://docs.sonarqube.org/latest/setup/get-started-2-minutes/

https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/

https://docs.sonarqube.org/latest/analysis/gitlab-integration/