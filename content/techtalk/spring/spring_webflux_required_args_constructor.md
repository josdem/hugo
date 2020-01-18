+++
title =  "Spring Webflux Constructor Injection"
description = "Constructor injection using Lombok in a Spring Webflux application"
date = "2020-01-18T17:38:44-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

[Lombok](https://projectlombok.org/) is a great tool to avoid boilerplate code, this time we will show you how to do constructor injection using Lombok in a [Spring Webflux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) application. If you want to know more about how to create Spring Webflux please go to my previous post getting started with Spring Webflux [here](/techtalk/spring/spring_webflux_basics). Then, let’s create a new Spring Boot project with Webflux as dependencies:

```bash
spring init --dependencies=webflux,lombok --build=gradle --language=java spring-webflux-required-args-constructor
```

Here is the complete `build.gradle` file generated:

```groovy
plugins {
	id 'org.springframework.boot' version '2.2.2.RELEASE'
	id 'io.spring.dependency-management' version '1.0.8.RELEASE'
	id 'java'
}

group = 'com.jos.dem.spring.webflux.lombok'
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

Now please consider this PersonController

```java
package com.jos.dem.spring.webflux.lombok.controller;

import com.jos.dem.spring.webflux.lombok.model.Person;
import com.jos.dem.spring.webflux.lombok.service.PersonService;
import lombok.RequiredArgsConstructor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@RestController
@RequestMapping("/persons")
@RequiredArgsConstructor
public class PersonController {

  private final PersonService personService;

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @GetMapping("/")
  public Flux<Person> findAll(){
    log.info("Calling find persons");
    return personService.getAll();
  }

  @GetMapping("/{nickname}")
  public Mono<Person> findById(@PathVariable String nickname){
    log.info("Calling find person by nickname: {}", nickname);
    return personService.getByNickname(nickname);
  }

}
```

That's it, with `@RequiredArgsConstructor` we are avoiding to use `@Autowired private PersonService personSerivice;` and with that action we have a code more clean and easy to test. This is the controller test case:

```java
package com.jos.dem.spring.webflux.lombok;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertTrue;

import com.jos.dem.spring.webflux.lombok.model.Person;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.test.web.reactive.server.WebTestClient;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class PersonControllerTest {

	@Autowired
	private WebTestClient webTestClient;

	@Test
	@DisplayName("Should get all persons")
	void shouldGetAllPersons() {
		webTestClient.get()
				.uri("/persons/")
				.exchange()
				.expectStatus().isOk()
				.expectBodyList(Person.class)
				.value(persons -> {
					assertTrue(persons.contains(new Person("josdem", "joseluis.delacruz@gmail.com")), "should contain josdem");
					assertTrue(persons.contains(new Person("tgrip", "tgrip@email.com")), "should contain tgrip");
					assertTrue(persons.contains(new Person("edzero", "edzero@email.com")), "should contain edzero");
					assertTrue(persons.contains(new Person("skuarch", "skuarch@email.com")), "should contain skuarch");
					assertTrue(persons.contains(new Person("jeduan", "jeduan@email.com")), "should contain jeduan");
				});
	}

	@Test
	@DisplayName("Should get josdem")
	void shouldGetPerson(){
		webTestClient.get()
				.uri("/persons/josdem")
				.exchange()
				.expectStatus().isOk()
				.expectBody(Person.class)
				.value(person -> {
					assertEquals(new Person("josdem", "joseluis.delacruz@gmail.com"), person, "should get josdem");
				});
	}

}
```

This is our `PersonService`

```java
package com.jos.dem.spring.webflux.lombok.service;

import com.jos.dem.spring.webflux.lombok.model.Person;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

public interface PersonService {

  Flux<Person> getAll();
  Mono<Person> getByNickname(String nickname);

}
```

And this the implementation:

```java
package com.jos.dem.spring.webflux.lombok.service.impl;

import com.jos.dem.spring.webflux.lombok.model.Person;
import com.jos.dem.spring.webflux.lombok.service.PersonService;
import java.util.HashMap;
import java.util.Map;
import java.util.stream.Stream;
import javax.annotation.PostConstruct;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@Service
public class PersonServiceImpl implements PersonService {

  private Map<String, Person> persons = new HashMap<>();

  @PostConstruct
  public void setup(){
    Stream.of(new Person("josdem", "joseluis.delacruz@gmail.com"),
        new Person("tgrip", "tgrip@email.com"),
        new Person("edzero", "edzero@email.com"),
        new Person("skuarch", "skuarch@email.com"),
        new Person("jeduan", "jeduan@email.com"))
        .forEach(person -> persons.put(person.getNickname(), person));
  }

  public Flux<Person> getAll(){
    return Flux.fromIterable(persons.values());
  }

  public Mono<Person> getByNickname(String nickname){
    return Mono.just(persons.get(nickname));
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

To browse the project go [here](https://github.com/josdem/spring-webflux-required-args-constructor), to download the project:

```bash
git clone git@github.com:josdem/spring-webflux-required-args-constructor.git
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
