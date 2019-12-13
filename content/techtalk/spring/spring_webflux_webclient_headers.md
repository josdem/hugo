+++
title =  "Spring Webflux Webclient Headers"
description = "In this technical post we will review how to read headers using WebClient"
date = "2019-07-23T10:24:17-04:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

HTTP headers allow the client and the server to pass additional information with the request or the response, if you want to know more about the list we can use as Http headers, please go [here](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields#Field_names). In this technical post we will see how to validate a server response including their headers using [WebTestClient](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/web/reactive/server/WebTestClient.html). If you want to know more about how to create Spring Webflux please go to my previous post getting started with Spring Webflux [here](/techtalk/spring/spring_webflux_basics). Then, let’s create a new Spring Boot project with Webflux as dependencies:

```bash
spring init --dependencies=webflux,lombok --language=java --build=gradle spring-webflux-webclient-workshop
```

Here is the complete `build.gradle` file generated:

```groovy
plugins {
  id 'org.springframework.boot' version '2.2.2.RELEASE'
  id 'io.spring.dependency-management' version '1.0.8.RELEASE'
  id 'java'
}

group = 'com.jos.dem.spring.webflux.webclient'
version = '1.0.0-SNAPSHOT'
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
  compileOnly('org.projectlombok:lombok')
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

Let's start by creating a controller to retrieve a basic response, a hello world concept :)

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

Let's create a person collection so that we can have some data in our `PersonRepository`

```java
package com.jos.dem.spring.webflux.webclient;

import com.jos.dem.spring.webflux.webclient.model.Person;
import com.jos.dem.spring.webflux.webclient.repository.PersonRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import java.util.Arrays;

@SpringBootApplication
public class DemoApplication {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }

  @Bean
  CommandLineRunner start(PersonRepository personRepository){
    return args -> {

      Arrays.asList(
          new Person("josdem", "joseluis.delacruz@gmail.com"),
          new Person("tgrip", "tgrip@email.com"),
          new Person("edzero", "edzero@email.com"),
          new Person("siedrix", "siedrix@email.com"),
          new Person("mkheck", "mkheck@losheckler.com"))
        .forEach(person -> personRepository.save(person));

    };
  }

}
```

It is time to create our test case in order to verify our behaviour.

```java
package com.jos.dem.spring.webflux.webclient;

import com.jos.dem.spring.webflux.webclient.model.Person;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.http.MediaType;
import org.springframework.test.web.reactive.server.WebTestClient;

import java.util.Date;

import static org.hamcrest.Matchers.equalTo;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class PersonControllerTest {

  @Autowired
  private WebTestClient webTestClient;

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Test
  @DisplayName("Should get all of persons")
  public void shouldGetAllPersons() throws Exception {
    log.info("Running: Should get all persons at {}", new Date());

    webTestClient.get().uri("/persons/")
            .exchange()
            .expectStatus().isOk()
            .expectHeader().contentType(MediaType.APPLICATION_JSON_VALUE)
            .expectBodyList(Person.class)
            .value(persons -> persons.size(), equalTo(5))
            .value(persons -> persons.contains(new Person("josdem","joseluis.delacruz@gmail.com")))
            .value(persons -> persons.contains(new Person("tgrip", "tgrip@email.com")))
            .value(persons -> persons.contains(new Person("edzero", "edzero@email.com")))
            .value(persons -> persons.contains(new Person("siedrix", "siedrix@email.com")))
            .value(persons -> persons.contains(new Person("mkheck", "mkheck@losheckler.com")));
  }

}
```

That's it, we are validating HttpHeaders using `expectHeader().contentType(MediaType.APPLICATION_JSON_VALUE)` if you want to see the complete list of validations we can do using Spring header assertions, please go [here](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/web/reactive/server/HeaderAssertions.html).

To run the project:

```bash
gradle bootRun
```

To test the project:

```bash
gradle test
```

To browse the project go [here](https://github.com/josdem/spring-webflux-webclient-workshop), to download the project:

```bash
git clone git@github.com:josdem/spring-webflux-webclient-workshop.git
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
