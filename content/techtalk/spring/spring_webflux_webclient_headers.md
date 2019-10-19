+++
title =  "Spring Webflux Webclient Headers"
description = "Spring_webflux_webclient_headers"
date = "2019-07-23T10:24:17-04:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

HTTP headers allow the client and the server to pass additional information with the request or the response, if you want to know more about the list we can use as Http headers, please go [here](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields#Field_names). In this technical post we will see how to validate a server response including their headers using [WebClient](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-webclient.html). If you want to know more about how to create Spring Webflux please go to my previous post getting started with Spring Webflux [here](/techtalk/spring/spring_webflux_basics). Then, let’s create a new Spring Boot project with Webflux as dependencies:

```bash
spring init --dependencies=webflux --language=java --build=gradle spring-webflux-webclient-workshop
```

Here is the complete `build.gradle` file generated:

```groovy
plugins {
  id 'org.springframework.boot' version '2.2.0.RELEASE'
  id 'io.spring.dependency-management' version '1.0.8.RELEASE'
  id 'java'
}

group = 'com.jos.dem.spring.webflux.webclient'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
  mavenCentral()
}

dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-webflux'
  testImplementation('org.springframework.boot:spring-boot-starter-test') {
    exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
  }
  testImplementation 'io.projectreactor:reactor-test'
}

test {
  useJUnitPlatform()
}
```

Let's start by creating a controller to retrieve a basic response, a hello world concept :)

```java
package com.jos.dem.spring.webflux.webclient.controller;

import reactor.core.publisher.Mono;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

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

Now let's define a `Webclient` as a `@Bean`, With webclient you can use both worlds blocking and non-blocking HTTP requests.

```java
package com.jos.dem.spring.webflux.webclient;

import org.springframework.boot.SpringApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.reactive.function.client.WebClient;

@SpringBootApplication
public class DemoApplication {

  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }

  @Bean
  WebClient webClient() {
    return WebClient.create("http://localhost:8080");
  }

}
```

Next step is to define a service with our WebClient request


```java
package com.jos.dem.spring.webflux.webclient.service;

import reactor.core.publisher.Mono;
import org.springframework.http.HttpHeaders;

public interface WebclientService {

  Mono<String> getGreetings();
  Mono<HttpHeaders> getHeaders();

}
```

In this example we are defining two metods in `getGreetings()` we expect to have our "HelloWorld!" message in the other method we expect to have our headers, here is our service implementation

```java
package com.jos.dem.spring.webflux.webclient.service.impl;

import reactor.core.publisher.Mono;

import org.springframework.http.HttpHeaders;
import org.springframework.web.reactive.function.client.WebClient;

import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;
import com.jos.dem.spring.webflux.webclient.service.WebclientService;

@Service
public class WebclientServiceImpl implements WebclientService {

  @Autowired
  private WebClient webClient;

  public Mono<String> getGreetings(){
    return webClient.get()
      .uri("/")
      .retrieve()
    .bodyToMono(String.class);
  }

  public Mono<HttpHeaders> getHeaders(){
    return webClient.get()
		  .uri("/")
		  .exchange()
		  .map(response -> response.headers().asHttpHeaders());
  }

}
```

It is time to create our test case so that we can verify our service behaviour.

```java
package com.jos.dem.spring.webflux.webclient;

import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.Date;

import org.junit.jupiter.api.Test;

import org.springframework.http.HttpHeaders;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;

import com.jos.dem.spring.webflux.webclient.service.WebclientService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class WebfluxControllerTest {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Autowired
  private WebclientService webclientService;

  @Test
  public void shouldGetHelloWorld() throws Exception {
    log.info("Running: Should get hello world message at {}", new Date());
    String response = webclientService.getGreetings().block();
    assertEquals("Hello World!", response);
  }

  @Test
  public void shouldGetHeaders() throws Exception {
    log.info("Running: Should get headers at {}", new Date());
    HttpHeaders headers = webclientService.getHeaders().block();
    assertEquals("text/plain;charset=UTF-8", headers.getContentType().toString());
    assertEquals(12L, headers.getContentLength());
  }

}
```

That's it, we are validating HttpHeaders which is a data structure representing HTTP request, mapping String header names to its values.

To run the project:

```bash
gradle bootRun
```

To test the project:

```bash
gradle test
```

To browse the project go [here](https://github.com/josdem/spring-webflux-webclient-workshop), to download the project:

```bash
git clone git@github.com:josdem/spring-webflux-webclient-workshop.git
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
