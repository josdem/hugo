+++
title =  "Spring Boot Server-sent Events"
description = "In this technical post we will see how to integrate Sever-sent events in a Spring Webflux application."
date = "2019-04-25T08:22:13-04:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

In this technical post we will see how to integrate [Sever-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) in a Spring Webflux application. Please read this previous [Spring Webflux Basics](/techtalk/spring/spring_webflux_basics) before conitnue with this information. Then, let’s create a new Spring Boot project with Webflux and Lombok as dependencies:

```bash
spring init --dependencies=webflux,lombok --language=java --build=gradle spring-boot-sse
```

Here is the complete `build.gradle` file generated:

```groovy
plugins {
	id 'org.springframework.boot' version '2.1.4.RELEASE'
	id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.sse'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

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

Let's start by creating a controller to serve our stream data

```java
package com.jos.dem.springboot.sse.controller;

import reactor.core.publisher.Flux;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.springboot.sse.model.Command;
import com.jos.dem.springboot.sse.service.MessageService;

@RestController
public class MessageController {

  @Autowired
  private MessageService messageService;

  @GetMapping("/")
  public Flux<Command> index() {
    return messageService.getAll();
  }

}
```

Command is just an abstraction to represent our domain transfer object general properties

```java
package com.jos.dem.springboot.sse.model;

import java.io.Serializable;

public interface Command extends Serializable {}
```

And here is our message object

```java
package com.jos.dem.springboot.sse.model;

import java.time.Instant;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class MessageCommand implements Command {
  private String nickname;
  private String text;
  private Instant timestamp;
}
```

Next step is create a service which returns a `Flux` object with our data

```java
package com.jos.dem.springboot.sse.service;

import reactor.core.publisher.Flux;

import com.jos.dem.springboot.sse.model.Command;

public interface MessageService {
  Flux<Command> getAll();
}
```

Here is our implementation

```java
package com.jos.dem.springboot.sse.service.impl;

import java.util.List;
import java.util.Arrays;
import java.time.Instant;
import java.time.Duration;

import reactor.core.publisher.Flux;

import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.springboot.sse.model.Command;
import com.jos.dem.springboot.sse.model.MessageCommand;
import com.jos.dem.springboot.sse.service.MessageService;
import com.jos.dem.springboot.sse.util.MessageGenerator;

@Service
public class MessageServiceImpl implements MessageService {

  @Autowired
  private MessageGenerator messageGenerator;

  public Flux<Command> getAll() {
    return Flux.interval(Duration.ofSeconds(1))
      .map(this::generateComment)
      .flatMapIterable(it -> it);
  }

  private List<Command> generateComment(long interval) {
    Command command = new MessageCommand(
      "josdem",
      messageGenerator.generate(),
      Instant.now());
    return Arrays.asList(command);
  }

}
```

Where:

* `Flux.interval` Simulate data streaming every second
* `flatMapIterable` Flattens a stream of Iterables
* `generateComment` Generate a random message

Here is our message generator class

```java
package com.jos.dem.springboot.sse.util;

import java.util.List;
import java.util.Arrays;
import java.util.Random;

import org.springframework.stereotype.Component;

@Component
public class MessageGenerator {

  private List<String> messages = Arrays.asList(
    "Bonjour",
    "Hola",
    "Zdravstvuyte",
    "Salve",
    "Guten Tag",
    "Hello");

  private final Random random = new Random(messages.size());

  public String generate() {
    return messages.get(random.nextInt(messages.size()));
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

To run the project:

```bash
gradle bootRun
```

To browse the project go [here](https://github.com/josdem/spring-boot-sse), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-sse.git
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)

