+++
title =  "Leyendo Headers con WebClient"
description = "En este post tecnico veremos como leer headers usando Webclient"
date = "2019-07-23T10:24:17-04:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

Los headers HTTP permiten al servidor y al cliente intercambiar información adicional más allá del body, si quieres saber más acerca de la lista de los headers existentes Http, por favor ve [aquí](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields#Field_names). En este post técnico veremos como validar la respuesta de un servidor incluyendo los headers usando [WebClient](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-webclient.html). Si quieres saber más acerca de como crear una aplicación Spring Webflux por favor visita mi previo post empezando con Spring Webflux [aquí](/techtalk/spring/spring_webflux_basics). Entonces, crea una aplicación Spring Boot con Webflux como dependencia:

```bash
spring init --dependencies=webflux --language=java --build=gradle spring-webflux-webclient-workshop
```

Aquí está el `build.gradle` generado:

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

Vamos a empezar creando un controlador para obtener una respuesta básica, el concepto hello world! :)

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

Ahora, vamos a definir un `Webclient` como un `@Bean`, Con webclient puedes usar ambos mundos peticiones bloqueantes y no bloqueantes.

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

El siguiente paso es definir un servicio con nuestra petición usando WebClient


```java
package com.jos.dem.spring.webflux.webclient.service;

import reactor.core.publisher.Mono;
import org.springframework.http.HttpHeaders;

public interface WebclientService {

  Mono<String> getGreetings();
  Mono<HttpHeaders> getHeaders();

}
```

En este ejemplo estamos definiendo dos métodos `getGreetings()` esperamos tener un mensaje de texto  "HelloWorld!" en el otro método esperamos obtener los headers, aquí está la implementación del servicio:

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

Es tiempo de crear nuestro test case, así podemos verificar el comportamiento de nuestro servicio.

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

Así es, estamos validando los HttpHeaders los cuales es una estrúctura de datos representando una petición HTTP, mapeando String como nombres del header y sus valores.

Para correr el proyecto:

```bash
gradle bootRun
```

Para testear el proyecto:

```bash
gradle test
```
Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/spring-webflux-webclient-workshop), para descargar el proyecto:

```bash
git clone git@github.com:josdem/spring-webflux-webclient-workshop.git
```


[Regresar al artículo principal](/techtalk/spring#Spring_Boot_Reactive_ES)
