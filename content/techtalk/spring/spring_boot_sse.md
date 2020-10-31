+++
title =  "Spring Boot Server-sent Events"
description = "In this technical post we will see how to integrate Sever-sent events in a Spring Webflux application."
date = "2019-04-25T08:22:13-04:00"
tags = ["josdem", "techtalks","programming","technology", "SSE example", "Spring Boot and SSE", "SSE with Spring Boot"]
categories = ["techtalk", "code"]
+++

In this technical post we will see how to integrate [Sever-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) in a Spring Webflux application. Please read this previous [Spring Webflux Basics](/techtalk/spring/spring_webflux_basics) before conitnue with this information. Then, let’s create a new Spring Boot project with Webflux and Lombok as dependencies:

```bash
spring init --dependencies=webflux,lombok --language=java --build=gradle spring-boot-sse
```

Here is the complete `build.gradle` file generated:

```groovy
plugins {
  id 'org.springframework.boot' version '2.3.4.RELEASE'
  id 'io.spring.dependency-management' version '1.0.10.RELEASE'
  id 'java'
}

group = 'com.jos.dem.springboot.sse'
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
}

test {
  useJUnitPlatform()
}
```

Let's start by creating a controller to serve our stream data

```java
package com.jos.dem.springboot.sse.controller;

import lombok.RequiredArgsConstructor;
import reactor.core.publisher.Flux;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import com.jos.dem.springboot.sse.model.Event;
import com.jos.dem.springboot.sse.service.MessageService;

@RestController
@RequiredArgsConstructor
public class MessageController {

  private final MessageService messageService;

  @GetMapping(path = "/",  produces = MediaType.TEXT_EVENT_STREAM_VALUE)
  public Flux<Event> index() {
    return messageService.stream();
  }

}
```

`MediaType.TEXT_EVENT_STREAM_VALUE` represents plain text event sent that follows SSE format, responses starts with `"data:"`. Now let's use `Event` as domain transfer object

```java
package com.jos.dem.springboot.sse.model;

import java.time.Instant;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Event {
  private String nickname;
  private String text;
  private Instant timestamp;
}
```

Next step is create a service which returns a `Flux` object with our data

```java
package com.jos.dem.springboot.sse.service;

import reactor.core.publisher.Flux;

import com.jos.dem.springboot.sse.model.MessageCommand;

public interface MessageService {
  Flux<MessageCommand> stream();
}
```

Here is our implementation

```java
package com.jos.dem.springboot.sse.service.impl;

import java.time.Instant;
import java.time.Duration;

import lombok.RequiredArgsConstructor;
import reactor.core.publisher.Flux;

import org.springframework.stereotype.Service;

import com.jos.dem.springboot.sse.model.Event;
import com.jos.dem.springboot.sse.service.MessageService;
import com.jos.dem.springboot.sse.util.EventGenerator;

@Service
@RequiredArgsConstructor
public class MessageServiceImpl implements MessageService {

  private final EventGenerator eventGenerator;

  public Flux<Event> stream() {
    return Flux.interval(Duration.ofSeconds(1))
      .map(it -> new Event("josdem", eventGenerator.generate(), Instant.now()));
  }

}
```

Where:

* `Flux.interval` Simulate data streaming every second
* `eventGenerator` Generates a random message

Here is our message generator class

```java
package com.jos.dem.springboot.sse.util;

import java.util.List;
import java.util.Arrays;
import java.util.Random;

import org.springframework.stereotype.Component;

@Component
public class EventGenerator {

  private List<String> events = Arrays.asList(
    "Bonjour",
    "Hola",
    "Zdravstvuyte",
    "Salve",
    "Guten Tag",
    "Hello");

  private final Random random = new Random(events.size());

  public String generate() {
    return events.get(random.nextInt(events.size()));
  }

}
```

So, now if you execute our Spring Boot application

```bash
gradle bootRun
```

And hit our Spring Boot applicaiton end-point

```bash
curl http://localhost:8080/
```

Then you should see something similar to this output

```bash
data:{"nickname":"josdem","text":"Hola","timestamp":"2019-04-25T12:51:48.693987Z"}
data:{"nickname":"josdem","text":"Bonjour","timestamp":"2019-04-25T12:51:49.692895Z"}
data:{"nickname":"josdem","text":"Zdravstvuyte","timestamp":"2019-04-25T12:51:50.693260Z"}
data:{"nickname":"josdem","text":"Bonjour","timestamp":"2019-04-25T12:51:51.693231Z"}
data:{"nickname":"josdem","text":"Hello","timestamp":"2019-04-25T12:51:52.693324Z"}
```

Finally, let's use a Non-blocking, reactive client for testing our web layer, if you want to know more about WebTestClient, please go to my previous post [here](/techtalk/spring/spring_webflux_web_testing).

```java
package com.jos.dem.springboot.sse;

import com.jos.dem.springboot.sse.model.Event;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInfo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.http.MediaType;
import org.springframework.test.web.reactive.server.WebTestClient;

import java.util.List;

import static org.junit.jupiter.api.Assertions.assertEquals;

@Slf4j
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@RequiredArgsConstructor(onConstructor_ = @Autowired)
class ServerSentEventsClientApplicationTest {

  private final WebTestClient webClient;

  @Test
  @DisplayName("Should get five events")
  void shouldConsumeServerSentEvents(TestInfo testInfo) {
    log.info("Running: {}", testInfo.getDisplayName());

    List<Event> events = webClient.get().uri("/")
            .accept(MediaType.valueOf(MediaType.TEXT_EVENT_STREAM_VALUE))
            .exchange()
            .expectStatus().isOk()
            .returnResult(Event.class)
            .getResponseBody()
            .take(5)
            .collectList()
            .block();

    events.forEach(event -> log.info("event: {}", event));
    assertEquals(5, events.size());
  }

}
```

*NOTE:* Do not forget to add this lines to your `build.gradle` file so that you can use Lombok in test cases.

```java
testCompileOnly 'org.projectlombok:lombok'
testAnnotationProcessor "org.projectlombok:lombok"
```

To run the project:

```bash
gradle bootRun
```

To test the project:

```bash
gradle test
```


To browse the project go [here](https://github.com/josdem/spring-boot-sse), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-sse.git
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)

