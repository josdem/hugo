+++
date = "2015-08-29T20:12:17-05:00"
draft = true
title = "Spring Boot"

+++
## What is Spring Boot?
It is a Spring project aim to create easy web or stand-alone applications. It provides the next features:

* No requirement for XML configuration
* Tomcat or Jetty embedded
* Convention over configuration
* Easy setup
* You can create web, standalone, API or executable applications

This tutorial shows you how to create a simple Spring Boot project with this features:

* Groovy support
* Gradle construction
* Mysql databse support
* Basic configuration

You can create this project from command line like this:

```bash
spring init --dependencies=data-jpa,data-rest --build=gradle --language=groovy SimpleRestApplication
```

That command will generate a Spring project structure under SimpleRestApplication folder.

In order to do that you need to install [SDKMAN](http://sdkman.io/) if you are using Linux or Mac, or [posh-gvm](https://github.com/flofreud/posh-gvm) if you are using Windows. After that, you can easily install:

* Spring Boot
* Groovy
* Gradle

I will show you a short version of build.gradle generated.

```groovy
buildscript {
  ext {
    springBootVersion = '1.2.5.RELEASE'
  }
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  }
}

apply plugin: 'groovy'
apply plugin: 'idea'
apply plugin: 'spring-boot'

jar {
  baseName = 'demo'
  version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.7
targetCompatibility = 1.7

repositories {
  mavenCentral()
}

dependencies {
  compile("org.springframework.boot:spring-boot-starter-data-jpa")
  compile("org.springframework.boot:spring-boot-starter-data-rest")
  compile('org.codehaus.groovy:groovy:2.4.3')
  compile("mysql:mysql-connector-java:5.1.34")
  testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

This project will generate an DemoApplication.java file, rename it as DemoApplication.groovy in a package structure you want.

```groovy
package com.josdem

import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.SpringBootApplication

@SpringBootApplication
class DemoApplication {

  static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args)
  }
}
```

Next create a groovy file with simple rest controller description as follow:

```groovy
package com.josdem

import org.springframework.web.bind.annotation.RestController
import org.springframework.web.bind.annotation.RequestMapping

@RestController
class SimpleController {

  @RequestMapping("/")
  String index() {
    "Greetings from Spring Boot!"
  }

}
```

Mysql configuration keeps url connection, username and password description in the application.properties file.

```groovy
spring.datasource.url=jdbc:mysql://localhost/springboot
spring.datasource.username=josdem
spring.datasource.password=josdem
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

spring.jpa.generate-ddl=true
```

Finally run it from command line.

```
gradle bootRun
```

[Return to the main article](/techtalk/spring)
