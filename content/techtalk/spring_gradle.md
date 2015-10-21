+++
date = "2015-10-18T22:05:02-05:00"
draft = true
title = "Building with Gradle"

+++

Using DSL language you write your build script like you write your code and this is based on Groovy. The main advantages tu use Gradle are:

* Declarative Builds
* Build by convention
* It scales very well
* Multy-module projects
* Reusable pieces of build logic
* Intelligent tasks that can check if they need to be executed or not

In this post I will show you how to build Jmailer project using Gradle.

**Variables**

```groovy
def springVersion = '4.1.7.RELEASE'
def aspectjVersion = '1.8.7'
def groovyVersion = '2.4.5'
def jmsApiVersion = '1.1-rev-1'
def javassistVersion = '3.8.0.GA'
def hibernateValidatorVersion = '5.1.3.Final'
def currentEnvironment = project.hasProperty("environment")?environment:"development"
def jmailerConfigurationDir = "${System.getProperty('user.home')}/.jmailer"
```

Here we're defying variables in the top of the file in order to declare our dependencies. Notice that we can use Groovy to store user home path in the jmailerConfigurationDir.

**Subprojects**

In the next structure we are going to define a Groovy plugin, version, group package, and repositories notice that we're defying subprojects attributes.

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

As you can see at the :email:formatter bottom definition, it's an :emailer:sender dependency which means dependencies from sender are inherited to formatter.

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
}
```

Here we are defying war build, jetty server application, Java 8 compatibility, web project dependencies, context path and http port and a task which will perform properties replacement as follow:

* into is our path target directory to contains property files
* jmailerConfigurationDir is source folder, you can see it's definition at the top
* include is our file to copy in this task
* rename and replace do the trick to manage environments

To download the project

```bash
git clone git@github.com:josdem/jmailer-bootstrap.git
git fetch
git checkout feature/java-mail
```

[Return to the main article](/techtalk/spring)
