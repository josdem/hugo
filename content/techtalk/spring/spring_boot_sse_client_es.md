+++
title =  "Spring Webflux Server-sent Events el lado del Cliente"
description = "In this technical post we will see how to create a Sever-sent events client in a Spring Webflux application."
date = "2019-05-13T20:25:54-04:00"
tags = ["josdem", "techtalks","programming","technology","Webflux"]
categories = ["techtalk", "code"]
+++

In this technical post we will see how to consume an event stream using [Sever-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) in a Spring Webflux application. Please read this previous [Spring Boot Server-sent Events](/techtalk/spring/spring_boot_sse) before conitnue with this information. Then, let’s create a new Spring Boot project with Webflux and Lombok as dependencies:

```bash
spring init --dependencies=webflux,lombok --language=java --build=gradle spring-boot-sse-client
```

Here is the complete `build.gradle` file generated:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.5.RELEASE'
  id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.sse.client'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
  compileOnly {
    extendsFrom annotationProcessor
  }
}

repositories {
  mavenCentral()
}

dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-webflux'
  compileOnly 'org.projectlombok:lombok'
  annotationProcessor 'org.projectlombok:lombok'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
  testImplementation 'io.projectreactor:reactor-test'
}
```

Now add Junit 5 Framework dependencies to your `build.gradle` file:

```groovy
testCompile "org.junit.jupiter:junit-jupiter-api:$junitJupiterVersion"
testRuntime "org.junit.jupiter:junit-jupiter-engine:$junitJupiterVersion"
```

Let's start by creating a service to handle data consumption

```java
package com.jos.dem.springboot.sse.client.service;

import reactor.core.publisher.Flux;

import org.springframework.http.codec.ServerSentEvent;

public interface ServerSentEventsConsumerService {

  Flux<ServerSentEvent<String>> consume();

}
```

This is the service implementation

```java
package com.jos.dem.springboot.sse.client.service.impl;

import reactor.core.publisher.Flux;

import org.springframework.stereotype.Service;
import org.springframework.http.codec.ServerSentEvent;
import org.springframework.core.ParameterizedTypeReference;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.reactive.function.client.WebClient;

import com.jos.dem.springboot.sse.client.service.ServerSentEventsConsumerService;

@Service
public class ServerSentEventsConsumerServiceImpl implements ServerSentEventsConsumerService {

  @Autowired
  private WebClient webCLient;

  private ParameterizedTypeReference<ServerSentEvent<String>> type = new ParameterizedTypeReference<ServerSentEvent<String>>() {};

  public Flux<ServerSentEvent<String>> consume(){
    return webCLient.get()
      .uri("/")
      .retrieve()
      .bodyToFlux(type);
  }

}
```

The given `ParameterizedTypeReference` is used to pass generic type information. Here you can see how we are defining our `WebClient` as a Spring bean.

```java
package com.jos.dem.springboot.sse.client;

import org.springframework.context.annotation.Bean;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.reactive.function.client.WebClient;

@SpringBootApplication
public class ServerSentEventsClientApplication {

  public static void main(String[] args) {
    SpringApplication.run(ServerSentEventsClientApplication.class, args);
  }

  @Bean
  WebClient webClient() {
    return WebClient.create("http://localhost:8080/");
  }

}
```

Here we are defining a controller, so that we will be able to start the stream consumption

```java
package com.jos.dem.springboot.sse.client.controller;

import java.time.LocalTime;

import reactor.core.publisher.Flux;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.codec.ServerSentEvent;

import com.jos.dem.springboot.sse.client.service.ServerSentEventsConsumerService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@RestController
public class ServerSentEventsConsumerController {

  @Autowired
  private ServerSentEventsConsumerService service;

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @GetMapping("/")
  public String index() {
    Flux<ServerSentEvent<String>> eventStream = service.consume();

    eventStream.subscribe(ctx ->
      log.info("Current time: {}, content[{}] ", LocalTime.now(), ctx.data()));

    return "Starting to consume events...";
  }

}
```

The subscribe method keeps receiving information as long as the server is still returning data. That's a client subscribes to a stream from a server and the server will send messages type event-stream to the client until the server or the client closes the stream. It is up to the server to decide when and what to send the client. For knowing more about Server-sent events technology please go [here](https://en.wikipedia.org/wiki/Server-sent_events). Do not forget to run this client in different port using the following specification in our `application.properties` file

```properties
server.port=8081
```

Now, If you run the project:

```bash
gradle bootRun
```

And hit this end-point:

```bash
curl http://localhost:8081/
```

Then you should see something like this:

```bash
2019-05-13 21:41:36.088 INFO 9103 [ctor-http-nio-6] : Current time: 21:41:36.087150, content[{"nickname":"josdem","text":"Guten Tag","timestamp":"2019-05-14T01:41:36.031100Z"}]
2019-05-13 21:41:37.034 INFO 9103 [ctor-http-nio-6] : Current time: 21:41:37.034186, content[{"nickname":"josdem","text":"Zdravstvuyte","timestamp":"2019-05-14T01:41:37.030950Z"}]
2019-05-13 21:41:38.034 INFO 9103 [ctor-http-nio-6] : Current time: 21:41:38.034517, content[{"nickname":"josdem","text":"Bonjour","timestamp":"2019-05-14T01:41:38.030778Z"}]
2019-05-13 21:41:39.031 INFO 9103 [ctor-http-nio-6] : Current time: 21:41:39.031638, content[{"nickname":"josdem","text":"Salve","timestamp":"2019-05-14T01:41:39.029399Z"}]
2019-05-13 21:41:40.033 INFO 9103 [ctor-http-nio-6] : Current time: 21:41:40.033601, content[{"nickname":"josdem","text":"Hola","timestamp":"2019-05-14T01:41:40.030511Z"}]
```

To browse the project go [here](https://github.com/josdem/spring-boot-sse-client), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-sse-client.git
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)

