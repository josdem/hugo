+++
date = "2015-10-12T20:26:12-05:00"
draft = true
title = "Basic Spring MVC with Bootstrap"

+++

## Technologies

* Gradle 2.7
* Spring 4.1.7
* Java 8
* Groovy 2.4.5
* XML configuration

**build.gradle**

```bash
apply plugin: 'groovy'
apply plugin: 'war'
apply plugin: 'jetty'

sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
    mavenLocal()
    mavenCentral()
}

def springVersion = '4.1.7.RELEASE'
def hibernateValidatorVersion = "5.2.2.Final"
def groovyVersion = '2.4.5'
def aspectjVersion = '1.8.7'
def currentEnvironment = project.hasProperty("environment")?environment:"development"

dependencies {

  compile "org.springframework:spring-webmvc:${springVersion}"
  compile "org.codehaus.groovy:groovy:${groovyVersion}"
  compile "org.hibernate:hibernate-validator:${hibernateValidatorVersion}"
  compile 'log4j:log4j:1.2.17'

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

task settingLog4jProperties(type:Copy){
  from "${System.getProperty('user.home')}/.jmailer/log4j-${currentEnvironment}.properties"
  into "src/main/resources/"
  rename { String fileName -> fileName.replace("-${currentEnvironment}", '') }
}

processResources.dependsOn "settingLog4jProperties"
```

