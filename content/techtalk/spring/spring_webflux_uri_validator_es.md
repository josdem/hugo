+++
title =  "Validando con Spring Webflux"
description = "Este post tecnico muestra como validar usando path variable"
date = "2019-05-19T13:40:37-04:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

[Spring Framework Validated](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/validation/annotation/Validated.html) Está designado como una implementación de Spring para el estándar [Spring's JSR-30](https://beanvalidation.org/1.0/spec/#constraintsdefinitionimplementation-constraintdefinition-parameters-message) y nos permite validar expresiones y beans de nuestras aplicaciones. En este post técnico veremos como validar beans usando path variable y expresiones regulares junto con la anotación Spring Validated en una aplicación Spring Webflux. **Nota:** Si quieres saber que herramientas necesitas tener instaladas en tu computadora para poder familiarizarte con Spring Boot por favor visita mi previo post: [Spring Boot](/techtalk/spring/spring_boot). Entonces ejecuta este comando desde la terminal.

```bash
spring init --dependencies=webflux,lombok --language=java --build=gradle spring-webflux-uri-validator
```

Aquí está el `build.gradle` generado:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.9.RELEASE'
  id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.uri.encoding'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.11'

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

Ahora agregamos las dependencias Junit 5 y Apache commons lang en el archivo `build.gradle`:

```groovy
implementation('org.apache.commons:commons-lang3:3.8.1')
testImplementation "org.junit.jupiter:junit-jupiter-api:$junitJupiterVersion"
testRuntime "org.junit.jupiter:junit-jupiter-engine:$junitJupiterVersion"
```

Vamos a empezar creando un servicio para abstraer la petición al controlador.

```java
package com.jos.dem.springboot.uri.validator.service;

import reactor.core.publisher.Mono;

public interface WebClientService {

  Mono<String> send(String path);

}
```

Este es la implementación del servicio

```java
package com.jos.dem.springboot.uri.validator.service.impl;

import reactor.core.publisher.Mono;

import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.reactive.function.client.WebClient;

import com.jos.dem.springboot.uri.validator.service.WebClientService;

@Service
public class WebClientServiceImpl implements WebClientService {

  @Autowired
  private WebClient webClient;

  public Mono<String> send(String path) {
    return webClient.get()
      .uri(path)
      .retrieve()
    .bodyToMono(String.class);
  }

}
```

Este servicio recibe una variable como path para ser enviada y evaluada para el controlador.

```java
package com.jos.dem.springboot.uri.validator.controller;

import reactor.core.publisher.Mono;

import org.apache.commons.lang3.builder.ToStringBuilder;

import javax.validation.constraints.Pattern;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.validation.annotation.Validated;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Validated
@RestController
public class ValidatorController {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @GetMapping("/{mnemonic}")
  public Mono<String> index(
      @PathVariable
        @Pattern(
          regexp = "^\\d{4}[-]\\d{2}[-]\\d{2}[-]\\d{6}$",
          message = "Invalid mnemonic format") String mnemonic) {
    log.info("INDEX: Mneminc is valid");
    return Mono.just("valid");
  }

}
```

Donde:

* `regexp` Es una expresión regular para evaluar: `YYYY-MM-DD-` más seis dígitos, si quieres saber más acerca de expresiones regulares en Java, porfavor ve [aquí](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html)
* `@PathVariable` Estamos leyendo el mnemónico como una variable de path
* `@Validated` Con esta anotación estamos hábilitando la validación con Spring

[WebClient](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-client) Es una instancia bean que está definida en nuestra aplicación Spring.

```java
package com.jos.dem.springboot.uri.validator;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.reactive.function.client.WebClient;

@SpringBootApplication
public class UriValidatorApplication {

  public static void main(String[] args) {
    SpringApplication.run(UriValidatorApplication.class, args);
  }

  @Bean
  WebClient webClient() {
    return WebClient.create("http://localhost:8080");
  }

}
```

Finalmente, vamos a crear un test case para validar el mnemónico recibido

```java
package com.jos.dem.springboot.uri.validator;

import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.Date;
import java.time.LocalTime;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.springboot.uri.validator.service.WebClientService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@RunWith(SpringRunner.class)
@SpringBootTest
public class ValidatorTest {

  @Autowired
  private WebClientService service;

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Test
  public void shouldValidateMnemonic() throws Exception {
    log.info("Running: Validating mnemonic at {}", new Date());

    String path = "2019-05-19-888123";
    assertEquals("valid", service.send(path).block());
  }

}
```

Para ver el error en la descripción cuando estamos enviando un mnemónico inválido, agreguemos un exception handler al controlador

```java
package com.jos.dem.springboot.uri.validator.controller;

import reactor.core.publisher.Mono;

import org.apache.commons.lang3.builder.ToStringBuilder;

import javax.validation.constraints.Pattern;
import javax.validation.ConstraintViolationException;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.validation.annotation.Validated;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Validated
@RestController
public class ValidatorController {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @GetMapping("/{mnemonic}")
  public Mono<String> index(
      @PathVariable
        @Pattern(
          regexp = "^\\d{4}[-]\\d{2}[-]\\d{2}[-]\\d{6}$",
          message = "Invalid mnemonic format") String mnemonic) {
    log.info("INDEX: Mneminc is valid");
    return Mono.just("valid");
  }

  @ExceptionHandler(ConstraintViolationException.class)
  Mono<String> handleException(ConstraintViolationException cve){
    log.info("Violations: " + ToStringBuilder.reflectionToString(cve.getConstraintViolations()));
    cve.getConstraintViolations().forEach(v -> log.info(ToStringBuilder.reflectionToString(v)));
    return Mono.just(cve.getMessage());
  }

}
```

Para correr el proyecto:

```bash
gradle bootRun
```

Para testear el proyecto:

```bash
gradle test
```

Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/spring-webflux-uri-validator), para descargar el proyecto:


```bash
git clone git@github.com:josdem/spring-webflux-uri-validator.git
```


[Regresar al artículo principal](/techtalk/spring#Spring_Boot_Reactive_ES)

