+++
title =  "Spring Webflux WebSockets"
description = "Spring Webflux with WebSockets"
date = "2020-02-27T11:52:46-05:00"
tags = ["josdem", "techtalks","programming","technology","websockets and spring boot example","websockets webflux example"]
categories = ["techtalk", "code", "spring", "webflux", "websockets"]
+++

[WebFlux](https://docs.spring.io/spring-framework/docs/5.0.0.BUILD-SNAPSHOT/spring-framework-reference/html/web-reactive.html) contains support for reactive HTTP and WebSocket clients. [WebSockets](https://en.wikipedia.org/wiki/WebSocket) allows communication in both directions and this could happen simultaneusly, therefore WebSockets are the best way to deliver real time data bi-direactional using both text or binary content. This time we will review how to build a WebFlux application using WebSockets from both client and server. If you want to know more about how to create Spring Webflux please go to my previous post getting started with Spring Webflux [here](/techtalk/spring/spring_webflux_basics). Then, let’s create a new Spring Boot project with Webflux and Lombok as dependencies:

```bash
spring init --dependencies=webflux,lombok --build=gradle --language=java webflux-websocket-workshop
```

Here is the complete `build.gradle` file generated:

```groovy
plugins {
	id 'org.springframework.boot' version '2.4.0'
	id 'io.spring.dependency-management' version '1.0.10.RELEASE'
	id 'java'
}

group = 'com.jos.dem.webflux.websocket'
version = '1.0.0-SNAPSHOT'
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

In order to implement WebSockets we need to create a `WebSocketHandlerAdpater` so that we can map each `WebSocketHandler` we are planning to use.

```java
package com.jos.dem.webflux.websocket.config;

import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.Ordered;
import org.springframework.web.reactive.HandlerMapping;
import org.springframework.web.reactive.handler.SimpleUrlHandlerMapping;
import org.springframework.web.reactive.socket.WebSocketHandler;

import java.util.HashMap;
import java.util.Map;

@Configuration
@RequiredArgsConstructor
public class WebSocketConfig {

  private final WebSocketHandler webSocketHandler;

  @Bean
  public HandlerMapping handlerMapping() {
    Map<String, WebSocketHandler> map = new HashMap<>();
    map.put("/channel", webSocketHandler);

    SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
    mapping.setOrder(Ordered.HIGHEST_PRECEDENCE);
    mapping.setUrlMap(map);
    return mapping;
  }

}
```

And here is our `ReactiveWebSocketHandler` which is managing our WebSocket session and messaging exchange.

```java
package com.jos.dem.webflux.websocket.handler;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.jos.dem.webflux.websocket.model.Event;
import com.jos.dem.webflux.websocket.util.MessageGenerator;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.socket.WebSocketHandler;
import org.springframework.web.reactive.socket.WebSocketSession;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import javax.annotation.PostConstruct;
import java.time.Duration;
import java.time.Instant;

@Component
@RequiredArgsConstructor
public class ReactiveWebSocketHandler implements WebSocketHandler {

  private Flux<String> intervalFlux;
  private final ObjectMapper mapper;
  private final MessageGenerator messageGenerator;

  @PostConstruct
  private void setup() {
    intervalFlux = Flux.interval(Duration.ofSeconds(2)).map(it -> getEvent());
  }

  @Override
  public Mono<Void> handle(WebSocketSession session) {
    return session.send(
        session
            .receive()
            .map(webSocketMessage -> webSocketMessage.getPayloadAsText())
            .log()
            .map(message -> session.textMessage(message)));
  }

  private String getEvent() {
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

You are good to execute our Spring Boot application server side, keep in mind that messages will start to flow once you execute the client side, we will see in a bit how we can execute both at the same time:

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
import com.jos.dem.webflux.websocket.model.Person;
import org.springframework.stereotype.Component;
import org.springframework.web.reactive.socket.WebSocketHandler;
import org.springframework.web.reactive.socket.WebSocketMessage;
import org.springframework.web.reactive.socket.WebSocketSession;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

import javax.annotation.PostConstruct;
import java.time.Duration;
import java.time.Instant;
import java.util.Arrays;
import java.util.List;

@Component
public class ReactiveWebSocketHandler implements WebSocketHandler {

  private Flux<Person> personStream;
  private final ObjectMapper mapper = new ObjectMapper();

  @PostConstruct
  private void setup() {
    List<Person> persons =
        Arrays.asList(
            new Person("josdem", "josdem@email.com"),
            new Person("skye", "skye@email.com"),
            new Person("tgrip", "tgrip@email.com"),
            new Person("edzero", "edzero@email.com"),
            new Person("jeduan", "jeduan@email.com"));

    personStream = Flux.fromIterable(persons).delayElements(Duration.ofSeconds(1));
  }

  @Override
  public Mono<Void> handle(WebSocketSession session) {

    Mono<Void> send = session.send(Mono.just(session.textMessage(getMessage("starting"))));

    Mono<Void> sendMessages =
        session
            .send(personStream.map(message -> session.textMessage(message.toString())))
            .and(session.receive().map(WebSocketMessage::getPayloadAsText).log());

    return send.then(sendMessages);
  }

  private String getMessage(String message) {
    JsonNode node = mapper.valueToTree(new Event(message, Instant.now()));
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

So, now if you execute our client Spring Boot application:

```bash
gradle bootRun
```

You should see this output from client side:

```bash
2020-02-27 15:16:18.702  INFO 33092 --- [           main] o.s.b.web.embedded.netty.NettyWebServer  : Netty started on port(s): 8081
2020-02-27 15:16:18.705  INFO 33092 --- [           main] c.j.d.w.w.WebsocketClientApplication     : Started WebsocketClientApplication in 0.841 seconds (JVM running for 1.078)
2020-02-27 15:16:18.940  INFO 33092 --- [ctor-http-nio-2] reactor.Flux.Map.1                       : onSubscribe(FluxMap.MapSubscriber)
2020-02-27 15:16:18.941  INFO 33092 --- [ctor-http-nio-2] reactor.Flux.Map.1                       : request(unbounded)
2020-02-27 15:16:19.934  INFO 33092 --- [ctor-http-nio-2] reactor.Flux.Map.1                       : onNext({"message":"Hola","timestamp":{"nano":898843000,"epochSecond":1582834579}})
2020-02-27 15:16:20.897  INFO 33092 --- [ctor-http-nio-2] reactor.Flux.Map.1                       : onNext({"message":"Bonjour","timestamp":{"nano":896173000,"epochSecond":1582834580}})
2020-02-27 15:16:21.897  INFO 33092 --- [ctor-http-nio-2] reactor.Flux.Map.1                       : onNext({"message":"Zdravstvuyte","timestamp":{"nano":896219000,"epochSecond":1582834581}})
2020-02-27 15:16:22.898  INFO 33092 --- [ctor-http-nio-2] reactor.Flux.Map.1                       : onNext({"message":"Bonjour","timestamp":{"nano":896291000,"epochSecond":1582834582}})
2020-02-27 15:16:23.899  INFO 33092 --- [ctor-http-nio-2] reactor.Flux.Map.1                       : onNext({"message":"Hola","timestamp":{"nano":898594000,"epochSecond":1582834583}})
2020-02-27 15:16:24.897  INFO 33092 --- [ctor-http-nio-2] reactor.Flux.Map.1                       : onNext({"message":"Hello","timestamp":{"nano":895673000,"epochSecond":1582834584}})
2020-02-27 15:16:25.900  INFO 33092 --- [ctor-http-nio-2] reactor.Flux.Map.1                       : onNext({"message":"Hello","timestamp":{"nano":896273000,"epochSecond":1582834585}})
2020-02-27 15:16:26.896  INFO 33092 --- [ctor-http-nio-2] reactor.Flux.Map.1                       : onNext({"message":"Guten Tag","timestamp":{"nano":895445000,"epochSecond":1582834586}})
2020-02-27 15:16:27.897  INFO 33092 --- [ctor-http-nio-2] reactor.Flux.Map.1                       : onNext({"message":"Guten Tag","timestamp":{"nano":896101000,"epochSecond":1582834587}})
2020-02-27 15:16:28.900  INFO 33092 --- [ctor-http-nio-2] reactor.Flux.Map.1                       : onNext({"message":"Hello","timestamp":{"nano":899134000,"epochSecond":1582834588}})
2020-02-27 15:16:29.565  INFO 33092 --- [ctor-http-nio-2] reactor.Flux.Map.1                       : onComplete()
2020-02-27 15:16:29.566  INFO 33092 --- [ctor-http-nio-2] ication$$EnhancerBySpringCGLIB$$76238f86 : complete
```

*Note:* Do not forget to add `server.port=8081` in application properties file, so we can run our client in a different port. Here you can find how another example about how to create a chatbot application using WebFlux, Websocket and JavaScript [WebFlux-WebSocket-Chatbot](https://github.com/josdem/webflux-websocket-chatbot)

**Using Maven**

You can do the same using Maven, the only difference is that you need to specify `--build=maven` parameter in the spring init command line:

```bash
spring init --dependencies=webflux,lombok --build=maven --language=java webflux-websocket-workshop
```

This is the `pom.xml` file generated

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.4.0</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.jos.dem.webflux.websocket</groupId>
    <artifactId>client</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <name>Spring Webflux WebSockets</name>
    <description>This project shows how to use Websocket with Spring Webflux</description>

    <properties>
        <java.version>13</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-test</artifactId>
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

To run the project with Gradle:

```bash
gradle bootRun
```

To run the project with Maven:

```bash
mvn spring-boot:run
```

To browse the project go [here](https://github.com/josdem/webflux-websocket-workshop), to download the project:

```bash
git clone git@github.com:josdem/webflux-websocket-workshop.git
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
