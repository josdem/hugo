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
* Groovy
* Gradle

Then execute this command in your terminal.

```bash
spring init --dependencies=web --language=groovy --build=gradle spring-boot-aop
```

This is the build.gradle file generated:

```groovy
buildscript {
	ext {
		springBootVersion = '1.5.9.RELEASE'
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

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}

dependencies {
	compile('org.springframework.boot:spring-boot-starter-web')
	compile('org.codehaus.groovy:groovy')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

## What is AOP?

Apply same logic in several points in your application is called cross cutting concerns, and in Spring we can implement it using AOP, some basics aspects are:

* Advice: Action taken. Different types of advice include “around,” “before”, “after”, "AfterReturning" and "AfterThrowing" advice.
* Pointcut: Where the action should be applied.
* Aspect: Is an advice and pointcut combination

In this example we are going to define an AfterThrowingAdvice which means an action is executed after some exception has been thrown, so first we need to define a general exception in our application.

```groovy
package com.jos.dem.springboot.aop.exception

class DemoException extends RuntimeException {

  DemoException(String message){
    super(message)
  }

  DemoException(String message, Throwable cause){
    super(message, cause)
  }

}
```

`DemoException` is a general exception in our application defined to react in the same way to the specific error.

So our advice look like this:

```groovy
package com.jos.dem.springboot.aop.aspect

import org.aspectj.lang.annotation.Aspect
import org.aspectj.lang.annotation.AfterThrowing
import org.springframework.stereotype.Component
import com.jos.dem.springboot.aop.exception.DemoException

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@Aspect
@Component
class AfterThrowingAdvice {

  Logger log = LoggerFactory.getLogger(this.class)

  @AfterThrowing(pointcut = "execution(* com.jos.dem.springboot.aop.service..**.*(..))", throwing = "ex")
  void doRecoveryActions(RuntimeException ex){
    log.info "Wrapping exception: ${ex}"
    throw new DemoException(ex.message, ex)
  }

}
```

**Explanation**

* An aspect is defined with the @Aspect annotation
* A bean managed by the spring context is defined with the @Component annotation
* A pointcut is defined as all methods in package com.jos.dem.jmailer.service with any parameter
* The action defined in the advice is thrown an `DemoException`

So if in any service we throw an Exception, actually a RuntimeException our advice logic is executed.


```groovy
package com.jos.dem.springboot.aop.service

interface DemoService {
  void show()
}

```

`DemoService` implementation

```groovy
package com.jos.dem.springboot.aop.service.impl

import org.springframework.stereotype.Service

import com.jos.dem.springboot.aop.service.DemoService
import com.jos.dem.springboot.aop.exception.DemoException

@Service
class DemoServiceImpl implements DemoService {

  void show(){
    throw new DemoException('Throwing Demo Exception')
  }

}
```

As you can see at `DemoService` in the `show()` method we are throwing an DemoException so our Advice will be called.

Here is the `DemoController` and a quick view how this controller is calling `DemoService`.

```groovy
package com.jos.dem.springboot.aop.controller

import org.springframework.web.bind.annotation.RestController
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.beans.factory.annotation.Autowired

import com.jos.dem.springboot.aop.service.DemoService

@RestController
class DemoController{

  @Autowired
  DemoService demoService

  @RequestMapping('/')
  String index(){
    demoService.show()
  }

}
```

To get this work, you only need to add aop dependency in your build.gradle

```bash
compile 'org.springframework.boot:spring-boot-starter-aop'
```

To download the project:

```bash
git clone https://github.com/josdem/spring-boot-aop.git
```

To run the project:

```bash
gradle bootRun
```

[Return to the main article](/techtalk/spring#Spring_Boot)
