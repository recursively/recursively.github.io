---
title: Sonar Code Quality Gate Integration with CI - Part 2
date: 2021-08-04 15:35:57
categories: SAST
tags: [sonarqube, sonar-scanner, Jenkins, Gitlab]
keywords: [sonarqube, sonar-scanner, Jenkins, Gitlab]
description: As Sonarqube and Gitlab CI integration described in the previous post, this time we're going to talk about the quality gate integration for Gitlab and Jenkins CI.
---
## Configuration for Jenkins and Gitlab

First and foremost, you have to configure the Jenkins CI and Gitlab to make sure they have permission to access each other. You can find the detailed guide to set them up properly from this website: https://docs.gitlab.com/ee/integration/jenkins.html#grant-jenkins-access-to-gitlab-project.

I used the webhook to notify Jenkins from Gitlab once any events are triggered. On the Jenkins, modify the project's configuration and generate a random secret token.

<img src="pic_1.png" width="100%" height="100%">

Fill in the blank URL area with your Jenkins server address in the Gtilab webhook configuration and paste the secret token to the next line.

<img src="pic_2.png" width="100%" height="100%">

After you finish the configuration you can take a test to verify the functionality. If everything goes well, you can get the status code 200 from Gitlab.

<img src="pic_3.png" width="100%" height="100%">

You can even go through the request and response packet to see what happened if you will.

<img src="pic_4.png" width="100%" height="100%">

The Jenkins job we just created is a pipeline job, which allows us to define our build tasks through the groovy script. There are some additional work you should do to allow Jenkins to trigger the build job from the *Jenkinsfile* from your repository.

<img src="pic_5.png" width="100%" height="100%">

## Tigger the Jenkins Pipeline

Create the *Jenkinsfile* file under the root directory of your repository, and add the build script:

```groovy
pipeline {
    agent any

    stages {
        stage('gitlab') {
            steps {
                echo 'Notify GitLab'
                updateGitlabCommitStatus name: 'build', state: 'pending'
                sh "make"
                updateGitlabCommitStatus name: 'build', state: 'success'
            }
        }
    }
}
```

The repository we're gonna use is the same as we created previously in this post: https://recursively.review/2021/07/28/Sonar-Code-Qualitygate-Integration-with-CI-Part-1/.

Commit your changes and push them to the remote repository to trigger the Jenkins pipeline. After a while, you can switch to Jenkins dashboard to check the build result.

<img src="pic_6.png" width="100%" height="100%">

## Integrate the Code Scanning

Let's first try using the cppcheck to perform the code scanning. This time we're gonna use the cppcheck plugin in Jenkins directly for convenience. Just install the cppcheck plugin and we're good to go.

```groovy
pipeline {
    agent any

    stages {
        stage('gitlab') {
            steps {
                echo 'Notify GitLab'
                updateGitlabCommitStatus name: 'build', state: 'pending'
                sh "make"
                updateGitlabCommitStatus name: 'build', state: 'success'
            }
        }
        stage('scan') {
            steps {
                echo 'Scan beginning'
                updateGitlabCommitStatus name: 'scan', state: 'running'
                sh "cppcheck --xml --xml-version=2 --enable=all ./ 2> cppcheck-report.xml"
                updateGitlabCommitStatus name: 'scan', state: 'success'
            }
        }
    }
}
```

Take a look at the Jenkins building dashboard to check the status.

<img src="pic_7.png" width="100%" height="100%">

Now that we have scanned our project successfully with cppcheck, it will not be difficult to integrate the Sonarqube in order to establish our quality gate. Before that, we need to install the Sonar-scanner plugin in Jenkins. When the installation is finished, go to **Manage Jenkins > Configure System** and scroll down to the **SonarQube servers** section. Click the **Add SonarQube** button to add the new configuration.

<img src="pic_8.png" width="100%" height="100%">

To use the Sonar-scanner command in the pipeline script, we have to firstly add a new Sonar-scanner tool in Jenkins.

<img src="pic_9.png" width="100%" height="100%">

## Quality Gate Integration

It's pretty easy to add the quality gate to our CI, let's make some changes to the sonar configuration file *sonar-project.properties*:

```ini
# must be unique in a given SonarQube instance
sonar.projectKey=test
sonar.login=50b94782744687df5d5b04863b6a3c2198b3361a
sonar.host.url=http://172.20.1.135:9000
sonar.qualitygate.wait=true

# --- optional properties ---

# defaults to project key
#sonar.projectName=My project
# defaults to 'not provided'
#sonar.projectVersion=1.0

# Path is relative to the sonar-project.properties file. Defaults to .
sonar.sources=.

#sonar.verbose=true

# Encoding of the source code. Default is default system encoding
#sonar.sourceEncoding=UTF-8

# mandatory: files to be handled by the _cxx plugin_
sonar.cxx.file.suffixes=.h,.cpp,.c
#sonar.cxx.errorRecoveryEnabled=True
#sonar.cxx.includeDirectories=./

# sonar.scm.exclusions.disabled=true

sonar.cxx.cppcheck.reportPaths=cppcheck-report.xml
```
For the *Jenkinsfile*:

```groovy
pipeline {
    agent any

    stages {
        stage('gitlab') {
            steps {
                echo 'Notify GitLab'
                updateGitlabCommitStatus name: 'build', state: 'pending'
                sh "make"
                updateGitlabCommitStatus name: 'build', state: 'success'
            }
        }
        stage('scan') {
            steps {
                echo 'Scan beginning'
                updateGitlabCommitStatus name: 'scan', state: 'running'
                sh "cppcheck --xml --xml-version=2 --enable=all ./ 2> cppcheck-report.xml"
                updateGitlabCommitStatus name: 'scan', state: 'success'
            }
        }
        stage('SonarQube analysis & quality gate') {
            environment {
                scannerHome = tool 'SonarScanner'
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
    }
}
```
Now that we have finished setting up the configuration regardingly. If we push our changes to the remote repository the CI quality gate check process will take effect.

<img src="pic_10.png" width="1000%" height="100%">

## Merge Request Combination

Firstly make some changes to the Jenkins pipeline script in order to modify the merge request status during the pipeline progress.

```groovy
pipeline {
    agent any

    stages {
        stage('gitlab') {
            steps {
                echo 'Notify GitLab'
                updateGitlabCommitStatus name: 'build', state: 'running'
                sh "make"
            }
        }
        stage('scan') {
            steps {
                echo 'Scan beginning'
                sh "cppcheck --xml --xml-version=2 --enable=all ./ 2> cppcheck-report.xml"
            }
        }
        stage('SonarQube analysis & quality gate') {
            environment {
                scannerHome = tool 'SonarScanner'
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
                updateGitlabCommitStatus name: 'build', state: 'success'
            }
        }
    }
}
```

To check the merge request scanning status, we need to enable the option below in the Gitlab:

<img src="pic_11.png" width="1000%" height="100%">

If the merge request was triggered, the merge request status will be limited unless the CI pipeline succeeds a moment later.

<img src="pic_12.png" width="1000%" height="100%">

## References

https://docs.gitlab.com/ee/integration/jenkins.html#grant-jenkins-access-to-gitlab-project

https://about.gitlab.com/handbook/customer-success/demo-systems/tutorials/integrations/create-jenkins-pipeline/