+++
date = "2018-01-15T15:45:00-06:00"
title = "Spring Boot Actuator"
categories = ["techtalk","code"]
tags = ["josdem","techtalks","programming","technology","Spring Boot Actuator"]

+++

Spring Boot includes a number of additional features to help you monitor and manage your application. The `spring-boot-actuator` module provides all of Spring Boot’s production-ready features.

Let’s start creating a new Spring Boot project with web and actuator dependencies:

```bash
spring init --dependencies=web,actuator --language=groovy --build=gradle spring-boot-actuator
```

This is the `build.gradle` file generated:

```groovy
buildscript {
  ext {
    springBootVersion = '1.5.12.RELEASE'
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

group = 'com.jos.dem.springboot.actuator'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
  mavenCentral()
}

dependencies {
  compile('org.springframework.boot:spring-boot-starter-web')
  compile('org.springframework.boot:spring-boot-starter-actuator')
  compile('org.codehaus.groovy:groovy')
  testCompile('org.springframework.boot:spring-boot-starter-test')
}

```

Now, let's create a `RestController`

```groovy
package com.jos.dem.springboot.actuator

import org.springframework.web.bind.annotation.RestController
import org.springframework.web.bind.annotation.RequestMapping

@RestController
class DemoController {

  @RequestMapping("/")
  String index(){
    'Hello World!'
  }

}
```

At this point you have an application Spring Boot running. Execute this command in order to see the Hello World message in [http://localhost:8080/](http://localhost:8080/)

```bash
gradle bootRun
```

Spring Boot Actuator enable you to monitor and manage your application, usually using HTTP end-points. You can monitor in terms of memory, uptime, threads, environment variables, beans created and more. Spring Boot Actuator defaults run on port 8080 as well. Let's change this in our `application.properties` file.

```properties
management.port: 9090
management.address: 127.0.0.1
```

By default Actuator end-points are protected by Spring Security. 

```json
{
  "timestamp": 1516053662310,
  "status": 401,
  "error": "Unauthorized",
  "message": "Full authentication is required to access this resource.",
  "path": "/metrics"
}
```

Let's deactivate Spring Security in order to simplify this example, to disable this you need to add this line in our `application.properties` file:


```properties
management.port: 9090
management.address: 127.0.0.1
management.security.enabled=false
```

After that changes let's run again our application

```bash
gradle bootRun
```

Now we are able to hit some Actuator's endpoints such as:

* http://localhost:9090/metrics
* http://localhost:9090/env
* http://localhost:9090/beans
* http://localhost:9090/loggers
* http://localhost:9090/dump

For a complete list please visit this official documentation: [Actuator Endpoints](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#production-ready-endpoints)

For example in metrics we can get this kind of information:

```json
{
  "mem": 296427,
  "mem.free": 206593,
  "processors": 4,
  "instance.uptime": 261456,
  "uptime": 270593,
  "systemload.average": -1,
  "heap.committed": 241664,
  "heap.init": 129024,
  "heap.used": 35070,
  "heap": 1815040,
  "nonheap.committed": 55784,
  "nonheap.init": 2496,
  "nonheap.used": 54765,
  "nonheap": 0,
  "threads.peak": 43,
  "threads.daemon": 36,
  "threads.totalStarted": 46,
  "threads": 39,
  "classes": 6499,
  "classes.loaded": 6499,
  "classes.unloaded": 0,
  "gc.ps_scavenge.count": 8,
  "gc.ps_scavenge.time": 143,
  "gc.ps_marksweep.count": 2,
  "gc.ps_marksweep.time": 225,
  "httpsessions.max": -1,
  "httpsessions.active": 0
}
```

We can add custom metrics, let's count how many times we are calling index method from our `DemoController`

```groovy
package com.jos.dem.springboot.actuator

import org.springframework.web.bind.annotation.RestController
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.boot.actuate.metrics.CounterService
import org.springframework.beans.factory.annotation.Autowired

@RestController
class DemoController {

  @Autowired
  CounterService counterService

  @RequestMapping("/")
  String index(){
  	counterService.increment("com.jos.dem.springboot.actuator.DemoController.index");
    'Hello World!'
  }

}
```

Now we can get this metrics version:

```json
{
  "mem": 296604,
  "mem.free": 203770,
  "processors": 4,
  "instance.uptime": 448569,
  "uptime": 457707,
  "systemload.average": -1,
  "heap.committed": 241664,
  "heap.init": 129024,
  "heap.used": 37893,
  "heap": 1815040,
  "nonheap.committed": 56104,
  "nonheap.init": 2496,
  "nonheap.used": 54940,
  "nonheap": 0,
  "threads.peak": 43,
  "threads.daemon": 36,
  "threads.totalStarted": 46,
  "threads": 39,
  "classes": 6520,
  "classes.loaded": 6520,
  "classes.unloaded": 0,
  "gc.ps_scavenge.count": 8,
  "gc.ps_scavenge.time": 143,
  "gc.ps_marksweep.count": 2,
  "gc.ps_marksweep.time": 225,
  "httpsessions.max": -1,
  "httpsessions.active": 0,
  "gauge.response.root": 36,
  "counter.status.200.root": 1,
  "counter.com.jos.dem.springboot.actuator.DemoController.index": 1
}
```

To download project:

```bash
git clone https://github.com/josdem/spring-boot-actuator.git
```

To run project:

```bash
gradle bootRun
```


[Return to the main article](/techtalk/spring)