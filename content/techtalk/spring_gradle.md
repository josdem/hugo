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

## Variables

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


