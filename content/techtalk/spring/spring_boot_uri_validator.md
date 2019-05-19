+++
title =  "Spring Webflux URI Validator"
description = "This technical post shows how to validate path variable formats"
date = "2019-05-19T13:40:37-04:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

[Hibernate Validator](http://hibernate.org/validator/) allows to express and validate application constraints, in this technical post we will see how to validate a path variable using Hibernate in a Spring Webflux application, if you want to know more about how to create Spring Webflux please go to my previous post getting started with Spring Webflux [here](/techtalk/spring/spring_webflux_basics). Then, let’s create a new Spring Boot project with Webflux and Lombok as dependencies:

```bash
spring init --dependencies=webflux,lombok --language=java --build=gradle spring-webflux-uri-validator
```

Here is the complete `build.gradle` file generated:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.5.RELEASE'
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

Now add Junit 5 and Apache commons lang dependencies to your `build.gradle` file:

```groovy
implementation('org.apache.commons:commons-lang3:3.8.1')
testImplementation "org.junit.jupiter:junit-jupiter-api:$junitJupiterVersion"
testRuntime "org.junit.jupiter:junit-jupiter-engine:$junitJupiterVersion"
```

Let's start by creating a service to send a request to the controller

```java
package com.jos.dem.springboot.uri.validator.service;

import reactor.core.publisher.Mono;

public interface WebClientService {

  Mono<String> send(String path);

}
```

This is the service implementation

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

This service implementation is receiving path as a variable and send it to a controller for validation.

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

Where:

* `regexp` Is a regular expresion to validate this format: `YYYY-MM-DD-` plus six digits, if you want to know more about regular expression in Java, please go [here](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html)
* `@PathVariable` We are reading mnemonic as path variable
* `@Validated` With this annotation we are enabling Hibernate validator

[WebClient](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html#webflux-client) is an bean instancem and it is defined in our validator Spring application

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

Finally, let's create a test case to validate our mnemonic value

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

If you want to see the error description when you are sending an invalid mnemonic, add an exception handler to the validator controller

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

To run the project:

```bash
gradle bootRun
```

To test the project:

```bash
gradle test
```

To browse the project go [here](https://github.com/josdem/spring-boot-uri-validator), to download the project:

```bash
git clone git@github.com:josdem/spring-webflux-uri-validator.git
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)

