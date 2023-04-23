+++
title =  "Spring Webflux Server"
categories = ["techtalk", "code","spring webflux"]
tags = ["josdem", "techtalks","programming","technology", "Reactive Programming", "WebFlux", "Reactor", "Java","spring webflux"]
date = "2018-03-26T16:10:59-06:00"
description = "This post walks you through the process of creating endpoints using Spring Webflux routers."
+++

This post walks you through the process of creating endpoints using Spring Webflux. Please read this previous [Spring Webflux Basics](/techtalk/spring/spring_webflux_basics) post before continuing with this information. We are going to use `@RestController` to serve data from our `PersonRepository`. Let’s consider the following example:

```java
package com.jos.dem.webflux.controller;

import reactor.core.publisher.Mono;
import reactor.core.publisher.Flux;

import lombok.RequiredArgsConstructor;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import com.jos.dem.webflux.model.Person;
import com.jos.dem.webflux.repository.PersonRepository;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@RestController
@RequiredArgsConstructor
public class PersonController {

    private final PersonRepository personRepository;

    @GetMapping("/persons")
    public Flux<Person> findAll() {
        log.info("Calling find persons");
        return personRepository.findAll();
    }

    @GetMapping("/persons/{nickname}")
    public Mono<Person> findById(@PathVariable String nickname) {
        log.info("Calling find person by id: {}", nickname);
        return personRepository.findById(nickname);
    }

}
```

The original Spring Web MVC was running on Tomcat, and the primary purpose was to run Servlet API and Servlet containers. The reactive web framework Spring WebFlux is fully non-blocking, supports Reactive Streams and runs on the Netty server. `@RestController` indicates that we don’t want to render views but write the results straight into the response body instead. `@GetMapping` tells Spring that this is the place to route person calls; it is the shorthand annotation for `@RequestMapping(method=RequestMethod.GET)`. So, after adding this controller class to your project and running Spring Boot, you can go to: http://localhost:8080/persons

```json
[
   {
      "nickname":"mkheck",
      "email":"mkheck@email.com"
   },
   {
      "nickname":"josdem",
      "email":"joseluis.delacruz@gmail.com"
   },
   {
      "nickname":"edzero",
      "email":"edzero@email.com"
   },
   {
      "nickname":"siedrix",
      "email":"siedrix@email.com"
   },
   {
      "nickname":"tgrip",
      "email":"tgrip@email.com"
   }
}
```

Here is the the endpoint to get a person by id: http://localhost:8080/persons/josdem

```json
{
  "nickname": "josdem",
  "email": "josdem@email.com"
}
```

**Testing The Web Layer**

The best tool for testing a reactive web server is [WebTestClient](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/web/reactive/server/WebTestClient.html) which is also a non-blocking, reactive client and it uses [WebClient](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-webclient.html) internally to perform requests and validate responses.

```java
package com.jos.dem.webflux;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.http.MediaType.APPLICATION_JSON_UTF8;

import lombok.extern.slf4j.Slf4j;
import lombok.RequiredArgsConstructor;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInfo;
import org.junit.jupiter.api.DisplayName;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.web.reactive.server.WebTestClient;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;

import com.jos.dem.webflux.model.Person;
import com.jos.dem.webflux.controller.PersonController;

@Slf4j
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@RequiredArgsConstructor(onConstructor_ = @Autowired)
class PersonControllerTest {

    private final WebTestClient webTestClient;

    @Test
    @DisplayName("getting all persons")
    void shouldGetPersons(TestInfo testInfo) {
        log.info("Running: {}", testInfo.getDisplayName());
        webTestClient.get().uri("/persons").accept(APPLICATION_JSON)
                .exchange()
                .expectStatus().isOk()
                .expectHeader().contentType(APPLICATION_JSON)
                .expectBodyList(Person.class);
    }

    @Test
    @DisplayName("getting person by nickname")
    void shouldGetPerson(TestInfo testInfo) {
        log.info("Running: {}", testInfo.getDisplayName());
        webTestClient.get().uri("/persons/{nickname}", "josdem").accept(APPLICATION_JSON)
                .exchange()
                .expectStatus().isOk()
                .expectHeader().contentType(APPLICATION_JSON)
                .expectBody(Person.class);
    }

}
```

**Conclusion**

Let me conclude by giving a short summary:

* Spring Boot now embraces reactive programming
* Using Spring MVC-style Controllers is a good start to adopting Reactive Web
* WebFlux works well with Java functional programming
* WebFlux runs on the Netty server by default
* The best tool for testing reactive web servers is to use WebTestClient

To run the project with Gradle:

```bash
gradle bootRun
```

To run the project with Maven:

```bash
mvn spring-boot:run
```

To test the project with Gradle:

```bash
gradle test
```

To test the project with Maven:

```bash
mvn test
```

To browse the project go [here](https://github.com/josdem/reactive-webflux-workshop), to download the project:

```bash
git clone https://github.com/josdem/reactive-webflux-workshop.git
cd server
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
