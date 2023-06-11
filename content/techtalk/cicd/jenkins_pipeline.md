+++
title =  "Jenkins Pipeline"
description = "In this technical post we will review basic concepts we need to know in order to create and execute Jenkins Pipeline"
date = "2019-02-27T08:00:31-05:00"
tags = ["josdem", "techtalks","programming","technology","ci","cd", "jenkins"]
categories = ["techtalk", "code","devops"]
+++

In this technical post, we will review concepts we need to know in order to create and execute [Jenkins Pipeline](https://jenkins.io/doc/book/pipeline/). A pipeline is an expression of our process to implement [Continuous Integration](https://en.wikipedia.org/wiki/Continuous_integration), we write our process using a [DSL](https://en.wikipedia.org/wiki/Domain-specific_language), and this is the basic structure using [Groovy](https://en.wikipedia.org/wiki/Apache_Groovy)

```groovy
#!/usr/bin/env groovy
pipeline {
  agent any
    stages {
      stage('build') {
        steps {
          echo 'Hello World!'
        }
      }
   }
}
```

where:

* `agent` Instructs Jenkins to allocate an executor
* `echo` Writes simple string in the console output
* `stage` A phase in our continuous integration process
* `steps` All steps within our stage

Now lets instruct to Jenkins to obtain your pipeline from Source Control Management (SCM) in this case [Github](https://github.com/)

![SCM definition](/img/techtalks/cicd/pipeline.png)

If you save it and click on the `Build Now` button you should be able to see this execution flow in the console output

```bash
Started by user josdem
Obtained Jenkinsfile from git https://github.com/josdem/pipeline-workshop.git
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
Running on Jenkins in /var/lib/jenkins/workspace/pipeline-sandbox
[Pipeline] // stage
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (build)
[Pipeline] echo
Hello World!
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS
```

Now, let's take a look at real project

```groovy
#!/usr/bin/env groovy
pipeline {
  agent any
  stages {
    stage('Stop Jugoterapia') {
      steps {
        echo 'Stoping Jugoterapia'
        sh 'sudo systemctl stop jugoterapia-webflux'
        echo 'Done!'
      }
    }
    stage ('Start Jugoterapia Webflux Job') {
      steps {
        echo 'Starting Jugoterapia Build Job'
        build job: 'jugoterapia-webflux'
        echo 'Done!'
      }
    }
    stage('Start Jugoterapia') {
      steps {
        echo 'Starting Jugoterapia'
        sh 'sudo systemctl start jugoterapia-webflux'
        echo 'Done!'
      }
    }
    stage('Start Jugoterapia Feature Test') {
      steps {
        echo 'Starting Jugoterapia Feature Testing'
        build job: 'jugoterapia-cucumber'
        echo 'Done!'
      }
    }
  }
}
```

where:

* `sh 'sudo systemctl stop jugoterapia-webflux'` Is a shell command to execute `systemctl stop` jugoterapia service
* `build job: 'jugoterapia-webflux'` Build another existing job in Jenkins named `jugoterapia-webflux`
* `sh 'sudo systemctl start jugoterapia-webflux'` Starts Jugoterapia service again.
* `build job: 'jugoterapia-cucumber'` Build another existing job for running feature testing


![Jugoterapia Pipeline](/img/techtalks/cicd/pipeline1.png)

Another important concept that it might be helpful to know is to write and read values using Groovy and keep them at the script level, you can do this like this:

```groovy
#!/usr/bin/env groovy
def userEmail
pipeline {
  agent any
    stages {
      stage('printing message') {
        steps {
          echo 'Hello World!'
        }
      }
      stage('storing value') {
        steps {
          script{
            userEmail = "contact@josdem.io"
          }
        }
      }
      stage('reading value') {
        steps {
          script {
            print userEmail
          }
        }
      }
   }
}
```

For more information how to execute shell commands or shell scripts using Jenkins, please go to my previous post [Jenkins Shell Execution](/techtalk/cicd/jenkins_shell_execution). To browse the project go [here](https://github.com/josdem/pipeline-workshop) to see the real project example, please go [here](https://github.com/josdem/jugoterapia-pipeline), to download the project:

```bash
git clone git@github.com:josdem/pipeline-workshop.git
```

And

```bash
git clone git@github.com:josdem/jugoterapia-pipeline.git
```


[Return to the main article](/techtalk/continuous_integration_delivery)


