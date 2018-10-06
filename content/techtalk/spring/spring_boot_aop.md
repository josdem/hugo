+++
title = "Spring Boot AOP"
date = "2015-11-29T21:25:06-06:00"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology","Spring Boot AOP", "spring boot"]
description = "This time I will show you how to create a basic project in Spring Boot using AOP"
+++

## Setup
This time I will show you how to create a basic project in Spring Boot using AOP, in order to do that you need to install [SDKMAN](http://sdkman.io/) if you are using Linux or Mac, or [posh-gvm](https://github.com/flofreud/posh-gvm) if you are using Windows. After that, you can easily install:

* Spring Boot
* Gradle

Then execute this command in your terminal.

```bash
spring init --dependencies=web --language=java --build=gradle spring-boot-aop
```

This is the build.gradle file generated:

```groovy
buildscript {
  ext {
    springBootVersion = '2.0.5.RELEASE'
  }
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  }
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.aop'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}

dependencies {
  compile('org.springframework.boot:spring-boot-starter-web')
  testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

## What is AOP?

Apply same logic in several points in your application is called cross cutting concerns, and in Spring we can implement it using AOP, some basics aspects are:

* Advice: Action taken. Different types of advice include “around,” “before”, “after”, "AfterReturning" and "AfterThrowing" advice.
* Pointcut: Where the action should be applied.
* Aspect: Is an advice and pointcut combination

In this example we are going to define an AfterThrowingAdvice which means an action is executed after some exception has been thrown, so first we need to define a general exception in our application.

```java
package com.jos.dem.springboot.aop.exception;

public class DemoException extends RuntimeException {

  public DemoException(String message){
    super(message);
  }

  public DemoException(String message, Throwable cause){
    super(message, cause);
  }

}
```

`DemoException` is a general exception in our application defined to react in the same way to the specific error.

So our advice look like this:

```java
package com.jos.dem.springboot.aop.aspect;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.AfterThrowing;
import org.springframework.stereotype.Component;
import com.jos.dem.springboot.aop.exception.DemoException;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Aspect
@Component
public class AfterThrowingAdvice {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @AfterThrowing(pointcut = "execution(* com.jos.dem.springboot.aop.service..**.*(..))", throwing = "ex")
  public void doRecoveryActions(RuntimeException ex){
    log.info("Wrapping exception: " + ex);
    throw new DemoException(ex.getMessage(), ex);
  }

}
```

**Explanation**

* An aspect is defined with the @Aspect annotation
* A bean managed by the spring context is defined with the @Component annotation
* A pointcut is defined as all methods in package `com.jos.dem.jmailer.service` with any parameter
* The action defined in the advice is thrown an `DemoException`

So if in any service we throw an Exception, actually a `RuntimeException` our advice logic is executed.


```java
package com.jos.dem.springboot.aop.service;

public interface DemoService {
  void show();
}
```

`DemoService` implementation

```java
package com.jos.dem.springboot.aop.service.impl;

import org.springframework.stereotype.Service;

import com.jos.dem.springboot.aop.service.DemoService;
import com.jos.dem.springboot.aop.exception.DemoException;

@Service
public class DemoServiceImpl implements DemoService {

  public void show(){
    throw new DemoException("Throwing Demo Exception");
  }

}
```

As you can see at `DemoService` in the `show()` method we are throwing an `DemoException` so our Advice will be called. Here is the `DemoController` and a quick view how this controller is calling `DemoService`.

```java
package com.jos.dem.springboot.aop.controller;

import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.springboot.aop.service.DemoService;

@RestController
public class DemoController {

  @Autowired
  private DemoService demoService;

  @RequestMapping("/")
  public String index(){
    demoService.show();
    return "Hello World!";
  }

}
```

To get this work, you only need to add aop dependency in your `build.gradle`

```bash
compile 'org.springframework.boot:spring-boot-starter-aop'
```

Finally, let's run this project executing: `gradle bootRun` command and hit the `http://localhost:8080/` url, then you should be able to see our after throwing advice aop strategy:

```bash
Wrapping exception: com.jos.dem.springboot.aop.exception.DemoException: Throwing Demo Exception
```

If you prefer Maven instead of Gradle you use this `pom.xml` definition:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem</groupId>
  <artifactId>aop</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-boot-aop</name>
  <description>Show How to use Spring Boot Aspect Oriented Programming</description>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.5.RELEASE</version>
    <relativePath/>
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-aop</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>

</project>
```

To browse the code go [here](https://github.com/josdem/spring-boot-aop), to download the code:

```bash
git clone git@github.com:josdem/spring-boot-aop.git
```

To run the project with Gradle:

```bash
gradle bootRun
```

To run the project with Maven:

```bash
mvn spring-boot:run
```

[Return to the main article](/techtalk/spring#Spring_Boot)
