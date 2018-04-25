+++
title = "Spring Boot Externalization"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology","Spring Boot","Spring Boot yaml"]
date = "2015-12-06T15:43:14-06:00"
description = "In this example we are going to externalize MySQL database connection using a yaml file. In order to get the setup for this example, please refer my previous post Spring Boot JPA"
+++

In this example we are going to externalize MySQL database connection using a yaml file. In order to get the setup for this example, please refer my previous post [Spring Boot JPA](/techtalk/spring/spring_boot_jpa)

First we need to create our yml file in ${USER_HOME}/.spring-boot-jpa/application-DEVELOPMENT.yml

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost/spring_boot_jpa
    username: username
    password: password
    driverClassName: com.mysql.jdbc.Driver
  jpa:
    generateDdl: true
```

Next we need to add bootRun closure and specify a system properties in our `build.gradle` file

```groovy
buildscript {
  ext {
    springBootVersion = '1.5.10.RELEASE'
  }
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  }
}

apply plugin: 'groovy'
apply plugin: 'org.springframework.boot'

group = 'com.jos.dem.springboot.jpa'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
  mavenCentral()
}


dependencies {
  compile('org.springframework.boot:spring-boot-starter-web')
  compile('org.springframework.boot:spring-boot-starter-thymeleaf')
  compile('org.springframework.boot:spring-boot-starter-data-jpa')
  compile("mysql:mysql-connector-java:5.1.34")
  compile('org.codehaus.groovy:groovy')
  testCompile('org.springframework.boot:spring-boot-starter-test')
}

bootRun {
  systemProperties = System.properties
}
```

That's it, now we can get our database credentials and connection from an externalized yaml file. In order to read our yaml file as system properties we need to execute this command in Mac or Linux:

```bash
gradle bootRun -Dspring.config.location=$HOME/.spring-boot-jpa/application-DEVELOPMENT.yml
```

If you are using Windows platform, use this command:

```bash
gradle bootRun "-Dspring.config.location=$HOME/.spring-boot-jpa/application-DEVELOPMENT.yml"
```

To download the project

```bash
git clone https://github.com/josdem/spring-boot-jpa.git
git fetch
git checkout feature/externalization
```

[Return to the main article](/techtalk/spring)
