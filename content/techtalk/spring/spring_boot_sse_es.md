+++
title =  "Spring Weblux y Server-sent Events"
description = "In this technical post we will see how to integrate Sever-sent events in a Spring Webflux application."
date = "2019-04-25T08:22:13-04:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

En este post técnico revisaremos como integrar [Sever-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) en una aplicación Spring Webflux. Por favor lee mi previo post [Spring Webflux Basics](/techtalk/spring/spring_webflux_basics) antes de continuar con esta información. Después, vamos a crear un nuevo proyecto Spring Boot con Webflux y Lombok como dependencias:

```bash
spring init --dependencies=webflux,lombok --language=java --build=gradle spring-boot-sse
```

Aquí está el `build.gradle` generado:

```groovy
plugins {
  id 'org.springframework.boot' version '2.2.0.RELEASE'
  id 'io.spring.dependency-management' version '1.0.8.RELEASE'
  id 'java'
}

group = 'com.jos.dem.springboot.sse'
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
  testImplementation('org.springframework.boot:spring-boot-starter-test') {
    exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
  }
  testImplementation 'io.projectreactor:reactor-test'
}

test {
  useJUnitPlatform()
}
```

Empecemos por crear un controlador para servir nuestro stream de datos.

```java
package com.jos.dem.springboot.sse.controller;

import reactor.core.publisher.Flux;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.springboot.sse.model.MessageCommand;
import com.jos.dem.springboot.sse.service.MessageService;

@RestController
public class MessageController {

  @Autowired
  private MessageService messageService;

  @GetMapping(path = "/",  produces = MediaType.TEXT_EVENT_STREAM_VALUE)
  public Flux<MessageCommand> index() {
    return messageService.stream();
  }

}
```

`MediaType.TEXT_EVENT_STREAM_VALUE` es necesario cuando quieres enviar server-sent events. Ahora vamos a usar `MessageCommand` como un objeto de transferencia de dominio.

```java
package com.jos.dem.springboot.sse.model;

import java.time.Instant;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class MessageCommand {
  private String nickname;
  private String text;
  private Instant timestamp;
}
```

El siguiente paseo es crear un servicio que regrese un objeto `Flux` con nuestros datos.

```java
package com.jos.dem.springboot.sse.service;

import reactor.core.publisher.Flux;

import com.jos.dem.springboot.sse.model.MessageCommand;

public interface MessageService {
  Flux<MessageCommand> stream();
}
```

Aquí está la implementación

```java
package com.jos.dem.springboot.sse.service.impl;

import java.util.List;
import java.util.Arrays;
import java.time.Instant;
import java.time.Duration;

import reactor.core.publisher.Flux;

import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.springboot.sse.model.MessageCommand;
import com.jos.dem.springboot.sse.service.MessageService;
import com.jos.dem.springboot.sse.util.MessageGenerator;

@Service
public class MessageServiceImpl implements MessageService {

  @Autowired
  private MessageGenerator messageGenerator;

  public Flux<MessageCommand> stream() {
    return Flux.interval(Duration.ofSeconds(1))
      .map(it -> new MessageCommand("josdem", messageGenerator.generate(), Instant.now()));
  }

}
```

Donde:

* `Flux.interval` Simula un stream de datos cada segundo
* `messageGenerator` Genera un mensaje random

Aquí está nuestra clase generadora de mensajes.

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

Así, ahora si ejecutamos nuestro proyecto Spring.

```bash
gradle bootRun
```

Y accesamos a nuestro principal end-point

```bash
curl http://localhost:8080/
```

Deberías ver algo similar a esta salida.

```bash
data:{"nickname":"josdem","text":"Hola","timestamp":"2019-04-25T12:51:48.693987Z"}
data:{"nickname":"josdem","text":"Bonjour","timestamp":"2019-04-25T12:51:49.692895Z"}
data:{"nickname":"josdem","text":"Zdravstvuyte","timestamp":"2019-04-25T12:51:50.693260Z"}
data:{"nickname":"josdem","text":"Bonjour","timestamp":"2019-04-25T12:51:51.693231Z"}
data:{"nickname":"josdem","text":"Hello","timestamp":"2019-04-25T12:51:52.693324Z"}
```

Finalmente, vamos a usar a un cliente reactivo y no-bloqueante para testear nuestro componente web, si quieres saber más acerca de WebTestClient, por favor visita mi previo post [aquí](/techtalk/spring/spring_webflux_web_testing).

```java
package com.jos.dem.springboot.sse;

import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.List;
import java.util.Date;
import java.time.LocalTime;

import reactor.core.publisher.Flux;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import org.springframework.http.MediaType;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.codec.ServerSentEvent;
import org.springframework.test.web.reactive.server.WebTestClient;

import com.jos.dem.springboot.sse.model.MessageCommand;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class ServerSentEventsClientApplicationTest {

  @Autowired
  private WebTestClient webClient;

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Test
  @DisplayName("Should get five events")
  void shouldConsumeServerSentEvents() {
    log.info("Running: Consume server sent events: {}", new Date());

    List<MessageCommand> commands = webClient.get().uri("/")
      .accept(MediaType.valueOf(MediaType.TEXT_EVENT_STREAM_VALUE))
      .exchange()
      .expectStatus().isOk()
      .returnResult(MessageCommand.class)
      .getResponseBody()
      .take(5)
      .collectList()
      .block();

    commands.forEach(command -> log.info("command: {}", command));
    assertEquals(5, commands.size());
  }

}
```

Para ejecutar  este proyecto:

```bash
gradle bootRun
```

Para testear el proyecto:

```bash
gradle test
```


Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/spring-boot-sse), para descargar el proyecto:

```bash
git clone git@github.com:josdem/spring-boot-sse.git
```

[Regresar al artículo principal](/techtalk/spring#Spring_Boot_Reactive_es))

