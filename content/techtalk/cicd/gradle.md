+++
title = "Building Software with Gradle"
categories = ["techtalk", "code","spring framework"]
tags = ["josdem", "techtalks", "programming", "technology","spring framework"]
date = "2015-10-18T22:05:02-05:00"
description = "Using DSL language you write your build script like you write your code and this is based on Groovy."
+++

Gradle is an open-source build automation tool focused on flexibility and performance. Gradle build scripts are written using a Groovy or Kotlin DSL. Using DSL language you write your build script file as you write your development code. Some advantages in use Gradle are:

* Acelerate development productivity
* Build by convention
* Automate everything
* Deliver faster persuing performance
* Multy-module projects
* Reusable pieces of build logic
* Intelligent tasks that can check if they need to be executed or not

In this post I will show you how to build software using simply but useful Gradle functionality. Firstly lets create a `build.gradle` file with a single task called helloWorld and add an action to it. The scripting language we are using here is Groovy, if you want to know more please go here: [Groovy](/techtalk/groovy/)

```groovy
task helloWorld {
  println "I am saying hello in the year of our lord: ${new Date()}"
}
```

Then execute this command in your terminal:

```bash
gradle helloWorld
```

*Output:*

```bash
I am saying hello in the year of our lord: Sun Oct 07 15:28:16 EDT 2018
```

**Subprojects**

In the next structure we are going to define a Groovy plugin, version, group package, repositories as subprojects attributes.

```groovy
subprojects {
  apply plugin:'groovy'
  version = "0.0.1"
  group = "com.jos.dem"

  repositories {
    mavenCentral()
  }
}
```

Now, we define two subprojects 'sender' and 'formatter' within emailer directory.

```groovy
project(":emailer:sender"){
  dependencies{
    compile "org.codehaus.groovy:groovy:$groovyVersion"
    compile 'org.springframework.integration:spring-integration-mail:4.2.0.RELEASE'
    compile "org.freemarker:freemarker:2.3.23"
    compile 'javax.mail:mail:1.4.7'
  }
}

project(":emailer:formatter"){
  dependencies{
    compile "javax.jms:jms-api:$jmsApiVersion"
    compile "org.hibernate:hibernate-validator:$hibernateValidatorVersion"
    compile 'org.apache.activemq:activemq-spring:5.12.0'
    compile project(":emailer:sender")
  }
}
```

As you can see at the :email:formatter bottom definition, it's an :emailer:sender dependency which means dependencies from sender are inherited.

**Main project**

Finally we defined our web project with the following structure.
```groovy
project(":web"){
  apply plugin: 'war'
  apply plugin: 'jetty'

  sourceCompatibility = 1.8
  targetCompatibility = 1.8

  dependencies {

    compile "org.springframework:spring-webmvc:$springVersion"
    compile "org.springframework:spring-jms:$springVersion"
    compile "org.aspectj:aspectjrt:$aspectjVersion"
    compile "org.aspectj:aspectjweaver:$aspectjVersion"
    compile 'com.google.code.gson:gson:2.2.2'
    compile 'log4j:log4j:1.2.17'
    compile 'javax.servlet:servlet-api:2.5'
    compile project(":emailer:formatter")
  }

  jettyRun{
    contextPath = "jmailer"
    httpPort = 8080
  }

  jettyRunWar{
    contextPath = "jmailer"
    httpPort = 8080
  }

  println "Setting environment to: ${currentEnvironment}"
  task settingEnvironment(type:Copy) {
    from jmailerConfigurationDir
    into 'src/main/resources/config'
    include "emailer-${currentEnvironment}.properties"
    rename { String fileName -> fileName.replace("-${currentEnvironment}", '') }
  }

  task settingLog4jProperties(type:Copy){
    from jmailerConfigurationDir
    into "src/main/resources/"
    include "log4j-${currentEnvironment}.properties"
    rename { String fileName -> fileName.replace("-${currentEnvironment}", '') }
  }

  processResources.dependsOn "settingEnvironment", "settingLog4jProperties"
}
```

Here we are defying war build, jetty server application, Java 8 compatibility, web project dependencies, context path, http port and two tasks, those are performing properties replacement as follow:

* **into** is our path target directory to contains property files
* **jmailerConfigurationDir** is source folder, you can see it's definition at the top
* **include** is our file to copy in this task
* **rename and replace** do the trick to manage environments


[Return to the main article](/techtalk/continuous_integration_delivery)
