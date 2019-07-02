+++
title =  "Spring Webflux Server-sent Events el lado del Cliente"
description = "In this technical post we will see how to create a Sever-sent events client in a Spring Webflux application."
date = "2019-05-13T20:25:54-04:00"
tags = ["josdem", "techtalks","programming","technology","Webflux"]
categories = ["techtalk", "code"]
+++

En este post técnico te mostraremos como consumir un stream de eventos usando [Sever-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) en una aplicación Spring Webflux. Por favor lee mis previo post [Spring Boot Server-sent Events](/techtalk/spring/spring_boot_sse) antes de continuar con esta información. Entonces, creamos un nuevo proyecto Spring Boot con Webflux y Lombok como dependencias:

```bash
spring init --dependencies=webflux,lombok --language=java --build=gradle spring-boot-sse-client
```

Aquí está el `build.gradle` generado:

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

Ahora por favor agrega la dependencias Junit 5 al archivo `build.gradle`:

```groovy
testCompile "org.junit.jupiter:junit-jupiter-api:$junitJupiterVersion"
testRuntime "org.junit.jupiter:junit-jupiter-engine:$junitJupiterVersion"
```

Empecemos por crear un servicio para manejar el consumo de datos.

```java
package com.jos.dem.springboot.sse.client.service;

import reactor.core.publisher.Flux;

import org.springframework.http.codec.ServerSentEvent;

public interface ServerSentEventsConsumerService {

  Flux<ServerSentEvent<String>> consume();

}
```

Esta es la implementación del servicio.

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

El valor `ParameterizedTypeReference` es usado para regresar valores de tipo genéricos. Aquí tu puedes ver como definimos nuestro `WebClient` como un bean de Spring.

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

Ahora definimos un controlador, de esta manera podemos iniciar el consumo del stream.

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

El método subscribe mantiene recibiendo información tanto como el servidor siga enviando datos. Así es un cliente se suscribe a un stream del servidor y el servidor envía mensajes de tipo event-stream al cliente hasta el servidor o el cliente cierra el stream. Es el control del servidor decidir cuando y como enviar al cliente. Para saner más acerca de la tgecnología Server-send events por favor ve [here](https://en.wikipedia.org/wiki/Server-sent_events). No olvides correr este cliente en un puerto diferente usando la siguiente especificación en el archbivo `application.properties`.

```properties
server.port=8081
```

Ahora, para correr el proyecto:

```bash
gradle bootRun
```

Y al consultar el end-point:

```bash
curl http://localhost:8081/
```

Deberías poder ver algo parecido a esto:

```bash
2019-05-13 21:41:36.088 INFO 9103 [ctor-http-nio-6] : Current time: 21:41:36.087150, content[{"nickname":"josdem","text":"Guten Tag","timestamp":"2019-05-14T01:41:36.031100Z"}]
2019-05-13 21:41:37.034 INFO 9103 [ctor-http-nio-6] : Current time: 21:41:37.034186, content[{"nickname":"josdem","text":"Zdravstvuyte","timestamp":"2019-05-14T01:41:37.030950Z"}]
2019-05-13 21:41:38.034 INFO 9103 [ctor-http-nio-6] : Current time: 21:41:38.034517, content[{"nickname":"josdem","text":"Bonjour","timestamp":"2019-05-14T01:41:38.030778Z"}]
2019-05-13 21:41:39.031 INFO 9103 [ctor-http-nio-6] : Current time: 21:41:39.031638, content[{"nickname":"josdem","text":"Salve","timestamp":"2019-05-14T01:41:39.029399Z"}]
2019-05-13 21:41:40.033 INFO 9103 [ctor-http-nio-6] : Current time: 21:41:40.033601, content[{"nickname":"josdem","text":"Hola","timestamp":"2019-05-14T01:41:40.030511Z"}]
```

Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/spring-boot-sse-clientp), para descargar el proyecto:

```bash
git clone git@github.com:josdem/spring-boot-sse-client.git
```


[Regresar al artículo principal](/techtalk/spring#Spring_Boot_Reactive_ES)

