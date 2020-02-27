+++
title =  "Spring Webflux WebSockets"
description = "Spring Webflux with WebSockets"
date = "2020-02-27T11:52:46-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code", "spring", "webflux", "websockets"]
+++

[WebFlux](https://docs.spring.io/spring-framework/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/html/web-reactive.html) contains support for reactive HTTP and WebSocket clients. [WebSockets](https://en.wikipedia.org/wiki/WebSocket) allows communication in both directions and this could happen simultaneusly, therefore WebSockets are the best way to deliver real time data bi-direactional using both text or binary content. This time we will review how to build a WebFlux application using WebSockets from both client and server. If you want to know more about how to create Spring Webflux please go to my previous post getting started with Spring Webflux [here](/techtalk/spring/spring_webflux_basics). Then, let’s create a new Spring Boot project with Webflux and Lombok as dependencies:

```bash
spring init --dependencies=webflux,lombok --build=gradle --language=java webflux-websocket-workshop
```

Here is the complete `build.gradle` file generated:

```groovy
plugins {
	id 'org.springframework.boot' version '2.2.4.RELEASE'
	id 'io.spring.dependency-management' version '1.0.9.RELEASE'
	id 'java'
}

group = 'com.jos.dem.webflux.websocket'
version = '1.0.0-SNAPSHOT'
sourceCompatibility = '12'

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

In order to implement WebSockets we need to create a `WebSocketHandlerAdpater` so that we can map each `WebSocketHandler` we are planning to use.

```java
package com.jos.dem.webflux.websocket;

import lombok.RequiredArgsConstructor;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.core.Ordered;
import org.springframework.web.reactive.HandlerMapping;
import org.springframework.web.reactive.handler.SimpleUrlHandlerMapping;
import org.springframework.web.reactive.socket.WebSocketHandler;
import org.springframework.web.reactive.socket.server.support.WebSocketHandlerAdapter;

import java.util.HashMap;
import java.util.Map;

@SpringBootApplication
@RequiredArgsConstructor
public class WebsocketApplication {

  private final WebSocketHandler webSocketHandler;

  public static void main(String[] args) {
    SpringApplication.run(WebsocketApplication.class, args);
  }

  @Bean
  public HandlerMapping handlerMapping() {
    Map<String, WebSocketHandler> map = new HashMap<>();
    map.put("/channel", webSocketHandler);

    SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
    mapping.setOrder(Ordered.HIGHEST_PRECEDENCE);
    mapping.setUrlMap(map);
    return mapping;
  }

  @Bean
  public WebSocketHandlerAdapter handlerAdapter() {
    return new WebSocketHandlerAdapter();
  }
}
```

And here is our `ReactiveWebSocketHandler` which is responsible of managing our WebSocket session.

```java
package com.jos.dem.webflux.websocket.handler;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.jos.dem.webflux.websocket.model.Event;
import com.jos.dem.webflux.websocket.util.MessageGenerator;
import java.time.Duration;
import java.time.Instant;
import javax.annotation.PostConstruct;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.socket.WebSocketHandler;
import org.springframework.web.reactive.socket.WebSocketMessage;
import org.springframework.web.reactive.socket.WebSocketSession;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@Component
@RequiredArgsConstructor
public class ReactiveWebSocketHandler implements WebSocketHandler {

  private Flux<String> intervalFlux;
  private final ObjectMapper mapper = new ObjectMapper();
  private final MessageGenerator messageGenerator;

  @PostConstruct
  private void setup(){
    intervalFlux =
        Flux.interval(Duration.ofSeconds(1)).map(it -> getEvent());
  }

  @Override
  public Mono<Void> handle(WebSocketSession session) {
    return session
        .send(intervalFlux.map(session::textMessage))
        .and(session.receive().map(WebSocketMessage::getPayloadAsText).log());
  }

  private String getEvent(){
    JsonNode node = mapper.valueToTree(new Event(messageGenerator.generate(), Instant.now()));
    return node.toString();
  }
}
```

Where:

* `Flux.interval` Simulate data streaming every second
* `handle` method is used here to send several texts as `WebSocketMessage` and receive a payload text back
* `messageGenerator` Generates a random message

Here is our message generator class

```java
package com.jos.dem.webflux.websocket.util;

import org.springframework.stereotype.Component;

import java.util.Arrays;
import java.util.List;
import java.util.Random;

@Component
public class MessageGenerator {

  private List<String> messages =
      Arrays.asList("Bonjour", "Hola", "Zdravstvuyte", "Salve", "Guten Tag", "Hello");

  private final Random random = new Random(messages.size());

  public String generate(){
      return messages.get(random.nextInt(messages.size()));
  }
}
```

To run the project:

```bash
gradle bootRun
```

**WebSockets Client Side**

Now, let's create a WebSocket client so that we will be able to connect to the server and exchange messages.

```java
package com.jos.dem.webflux.websocket.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.socket.client.ReactorNettyWebSocketClient;
import org.springframework.web.reactive.socket.client.WebSocketClient;

@Configuration
public class ReactiveWebSocketConfig {

  @Bean
  WebSocketClient webSocketClient() {
    return new ReactorNettyWebSocketClient();
  }
}
```

That's it, we are returning a bean as `ReactorNettyWebSocketClient` which is a `WebSocketClient` implementation. Here is our `ReactiveWebSocketHandler`

```java
package com.jos.dem.webflux.websocket.handler;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.jos.dem.webflux.websocket.model.Event;
import java.time.Instant;
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.socket.WebSocketHandler;
import org.springframework.web.reactive.socket.WebSocketMessage;
import org.springframework.web.reactive.socket.WebSocketSession;
import reactor.core.publisher.Mono;

@Component
public class ReactiveWebSocketHandler implements WebSocketHandler {

  private final ObjectMapper mapper = new ObjectMapper();

  @Override
  public Mono<Void> handle(WebSocketSession session) {
    return session
        .send(Mono.just(session.textMessage(getEvent())))
        .and(session.receive().map(WebSocketMessage::getPayloadAsText).log());
  }

  private String getEvent(){
    JsonNode node = mapper.valueToTree(new Event("start", Instant.now()));
    return node.toString();
  }
}
```

Where:

* `handle` method is used here to send a text as `WebSocketMessage` and receive several payloads as text back
* `getEvent` method formats an event bean as Json

Finally, we have our `WebSocketClientApplication` to start the conversation:

```java
package com.jos.dem.webflux.websocket;

import java.net.URI;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.reactive.socket.WebSocketHandler;
import org.springframework.web.reactive.socket.client.WebSocketClient;
import reactor.core.publisher.Mono;

@SpringBootApplication
public class WebsocketClientApplication {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  public static void main(String[] args) {
    SpringApplication.run(WebsocketClientApplication.class, args);
  }

  @Bean
  CommandLineRunner run(WebSocketClient client, WebSocketHandler webSocketHandler) {
    return args -> {
      Mono<Void> subscriber = client.execute(URI.create("ws://localhost:8080/channel"), webSocketHandler);

      subscriber.subscribe(
          content -> log.info("content"),
          error -> log.error("Error at receiving events: {}", error),
          () -> log.info("complete"));
      };
  }
}
```

To run the project:

```bash
gradle bootRun
```

*Note:* Do not forget to add `server.port=8081` in application properties file, so we can run our client in a different port. To browse the project go [here](https://github.com/josdem/webflux-websocket-workshop), to download the project:

```bash
git clone git@github.com:josdem/webflux-websocket-workshop.git
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
