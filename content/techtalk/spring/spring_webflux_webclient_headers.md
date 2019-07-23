+++
title =  "Spring Webflux Webclient Headers"
description = "Spring_webflux_webclient_headers"
date = "2019-07-23T10:24:17-04:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

HTTP headers allow the client and the server to pass additional information with the request or the response, if you want to know more about the list we can use as Http headers, please go [here](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields#Field_names). In this technical post we will see how to validate a server response including their headers using [WebClient](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-webclient.html). If you want to know more about how to create Spring Webflux please go to my previous post getting started with Spring Webflux [here](/techtalk/spring/spring_webflux_basics). Then, let’s create a new Spring Boot project with Webflux and Lombok as dependencies:

```bash
spring init --dependencies=webflux --language=java --build=gradle spring-webflux-webclient-workshop
```

Here is the complete `build.gradle` file generated:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.6.RELEASE'
  id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.spring.webflux.webclient'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
  mavenCentral()
}

dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-webflux'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
  testImplementation 'io.projectreactor:reactor-test'
}
```

Now add Junit 5 dependencies to your `build.gradle` file:

```groovy
testImplementation "org.junit.jupiter:junit-jupiter-api:$junitJupiterVersion"
testRuntime "org.junit.jupiter:junit-jupiter-engine:$junitJupiterVersion"
```

Let's start by creating a controller to retrieve a basic response, a hello world concept :)

```groovy
package com.jos.dem.spring.webflux.webclient;

import reactor.core.publisher.Mono;

import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.GetMapping;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@RestController
public class WebfluxController {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @GetMapping("/")
  public Mono<String> index(){
    log.info("Calling index");
    return Mono.just("Hello World!");
  }

}
```


