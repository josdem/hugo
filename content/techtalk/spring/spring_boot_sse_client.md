+++
title =  "Spring Boot Server-sent Events Client"
description = "In this technical post we will see how to create a Sever-sent events client in a Spring Webflux application."
date = "2019-05-13T20:25:54-04:00"
tags = ["josdem", "techtalks","programming","technology","Webflux", "Server Sent Events testing", "Server sent events client", "Server server events StepVerifier"]
categories = ["techtalk", "code"]
+++

In this technical post we will see how to consume events using [Sever-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) in a Spring Webflux application. Please read this previous [Spring Boot Server-sent Events](/techtalk/spring/spring_boot_sse) before conitnue with this information. Then, let’s create a new Spring Boot project with Webflux and Lombok as dependencies:

```bash
spring init --dependencies=webflux,lombok --language=java --build=gradle spring-boot-sse-client
```

Here is the complete `build.gradle` file generated:

```java
plugins {
  id 'org.springframework.boot' version '2.3.4.RELEASE'
  id 'io.spring.dependency-management' version '1.0.10.RELEASE'
  id 'java'
}

group = 'com.jos.dem.springboot.sse.client'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '13'

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
  testImplementation('org.springframework.boot:spring-boot-starter-test') {
    exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
  }
  testImplementation 'io.projectreactor:reactor-test'
  testCompileOnly 'org.projectlombok:lombok'
  testAnnotationProcessor "org.projectlombok:lombok"
}

test {
  useJUnitPlatform()
}
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

import com.jos.dem.springboot.sse.client.service.ServerSentEventsConsumerService;
import lombok.RequiredArgsConstructor;
import org.springframework.core.ParameterizedTypeReference;
import org.springframework.http.codec.ServerSentEvent;
import org.springframework.stereotype.Service;
import org.springframework.web.reactive.function.client.WebClient;
import reactor.core.publisher.Flux;

@Service
@RequiredArgsConstructor
public class ServerSentEventsConsumerServiceImpl implements ServerSentEventsConsumerService {

  private final WebClient webCLient;

  private ParameterizedTypeReference<ServerSentEvent<String>> type = new ParameterizedTypeReference<>() {
  };

  public Flux<ServerSentEvent<String>> consume(){
    return webCLient.get()
      .uri("/")
      .retrieve()
      .bodyToFlux(type);
  }

}
```

Here we are using `ParameterizedTypeReference` as generic type. Let's also create a `WebClient` bean definition.

```java
package com.jos.dem.springboot.sse.client.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.function.client.WebClient;

@Configuration
public class ApplicationConfig {
    @Bean
    WebClient webClient() {
        return WebClient.create("http://localhost:8080/");
    }
}
```

We are using `CommandRunner`, so that we can start the stream consumption. `CommandLineRunner` is a call back interface, when Spring Boot starts will call it and pass in args through a `run()` internal method.

```java
package com.jos.dem.springboot.sse.client;

import com.jos.dem.springboot.sse.client.service.ServerSentEventsConsumerService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.http.codec.ServerSentEvent;
import reactor.core.publisher.Flux;

import java.time.LocalTime;

@Slf4j
@SpringBootApplication
public class ServerSentEventsClientApplication {

  public static void main(String[] args) {
    SpringApplication.run(ServerSentEventsClientApplication.class, args);
  }

  @Bean
  CommandLineRunner start(ServerSentEventsConsumerService service) {
    return args -> {
      Flux<ServerSentEvent<String>> eventStream = service.consume();

      eventStream.subscribe(ctx ->
              log.info("Current time: {}, content[{}] ", LocalTime.now(), ctx.data()));
    };
  }

}
```

We are subscribing to the emiter in order to keep receiving information. That's it a client subscribes to a stream from a server and the server will send messages type event-stream to the client until the server or the client closes the stream. It is up to the server to decide when and what to send the client. For knowing more about Server-sent events technology please go [here](https://en.wikipedia.org/wiki/Server-sent_events). Do not forget to run this client in different port using the following specification in our `application.yml` file

```yaml
server:
  port: 8081
```

Finally, here you can find our test case to validate this functionality, `StepVerifier` is a tool from reactor that help us to do our testing in the reactive world.

```java
package com.jos.dem.springboot.sse.client;


import com.jos.dem.springboot.sse.client.service.ServerSentEventsConsumerService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInfo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.codec.ServerSentEvent;
import reactor.test.StepVerifier;

import java.time.Duration;

@Slf4j
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@RequiredArgsConstructor(onConstructor_ = @Autowired)
class ServerSentEventsClientApplicationTest {

    private final ServerSentEventsConsumerService service;

    @Test
    @DisplayName("Consume Server Sent Event")
    public void shouldConsumeServerSentEvents(TestInfo testInfo) throws Exception {
        log.info("Running: {}", testInfo.getDisplayName());

        StepVerifier.create(service.consume())
                .expectNextMatches(event -> event.data().contains("josdem"))
                .thenAwait(Duration.ofSeconds(5))
                .expectNextCount(1)
                .thenCancel()
                .verify();
    }

}
```

To run the project:

```bash
gradle bootRun
```

To test the project:

```bash
gradle bootRun
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

