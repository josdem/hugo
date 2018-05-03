+++
title =  "Spring Boot Cucumber"
categories = ["techtalk", "code","spring boot"]
tags = ["josdem", "techtalks","programming","technology","Java", "Cucumber","spring boot"]
date = "2018-03-30T19:42:17-06:00"
description = "BDD (Behavior-driven development) is a technique very similar to implement UAT (User Acceptance Testing) in a software project. Usually is a good idea to use BDD to reprecent how users can define application behaviour, so in that way you can represent user stories in test scenarios aka. feature testing."
+++

BDD (Behavior-driven development) is a technique very similar to implement UAT (User Acceptance Testing) in a software project. Usually is a good idea to use BDD to reprecent how users can define application behaviour, so in that way you can represent user stories in test scenarios aka. feature testing.

This time I am going to show you how integrate [Cucumber](https://cucumber.io/) to a Spring Boot application, Cucumber is a very powerful testing framework written in the Ruby programming language, which follows the BDD methodology. 

Let's start creating a new Spring Boot project with Webflux and Lombok as dependencies:

```bash
spring init --dependencies=webflux,lombok --build=gradle --language=java spring-boot-cucumber
```

Here is the complete `build.gradle` file generated:

```groovy
buildscript {
	ext {
		springBootVersion = '2.0.0.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.cucumber'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}

dependencies {
	compile('org.springframework.boot:spring-boot-starter-webflux')
	compileOnly('org.projectlombok:lombok')
	testCompile('org.springframework.boot:spring-boot-starter-test')
	testCompile('io.projectreactor:reactor-test')
}
```
Now let's create a simple POJO retrieve information from an end point using Spring WebFlux.

```java
package com.jos.dem.springboot.cucumber.model;

import lombok.AllArgsConstructor;
import lombok.NoArgsConstructor;
import lombok.ToString;
import lombok.Data;

@AllArgsConstructor
@ToString
@NoArgsConstructor
@Data
public class Person {

  private String nickname;
  private String email;

}
```

Lombok is a great tool to avoid boilerplate code, for knowing more please go [here](https://projectlombok.org/)

Next step is to create our routers to define `GET/persons` and `GET/persons` by nickname:

```java
package com.jos.dem.springboot.cucumber.config;

import static org.springframework.web.reactive.function.server.RequestPredicates.GET;

import com.jos.dem.springboot.cucumber.model.Person;
import com.jos.dem.springboot.cucumber.service.PersonService;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.function.server.RouterFunction;
import org.springframework.web.reactive.function.server.ServerResponse;
import org.springframework.web.reactive.function.server.RouterFunctions;

@Configuration
public class WebConfig {

  @Autowired
  private PersonService personService;

  @Bean
  public RouterFunction<ServerResponse> routes(){
    return RouterFunctions
      .route(GET("/persons"),
      request -> ServerResponse.ok().body(personService.getAll(), Person.class))
      .andRoute(GET("/persons/{nickname}"), 
      request -> ServerResponse.ok().body(personService.getByNickname(request.pathVariable("nickname")), Person.class));
  }

}
```

In order to complete our basement let's create `PersonService` to bring data:

```java
package com.jos.dem.springboot.cucumber.service;

import com.jos.dem.springboot.cucumber.model.Person;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

public interface PersonService {
  Flux<Person> getAll();
  Mono<Person> getByNickname(String nickname);
}
```

Implementation:

```java
package com.jos.dem.springboot.cucumber.service.impl;

import javax.annotation.PostConstruct;

import java.util.Map;
import java.util.Arrays;
import java.util.HashMap;
import java.util.stream.Stream;

import com.jos.dem.springboot.cucumber.model.Person;
import com.jos.dem.springboot.cucumber.service.PersonService;

import org.springframework.stereotype.Service;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@Service
public class PersonServiceImpl implements PersonService {

  private Map<String, Person> persons = new HashMap<String, Person>();

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

Now all is set in our service, let's continue adding Cucumber and Junit as dependencies in our `build.gradle`:

```groovy
testCompile('info.cukes:cucumber-java:1.2.5')
testCompile('info.cukes:cucumber-junit:1.2.5')
testCompile('info.cukes:cucumber-spring:1.2.5')
testCompile('junit:junit:4.12')
```

The JUnit runner uses the JUnit framework to run the Cucumber Test. What we need is to create a single empty class with an annotation `@RunWith(Cucumber.class)` and define `@CucumberOptions` where we’re specifying the location of the Gherkin file which is also known as the feature file:

```java
package com.jos.dem.springboot.cucumber;

import org.junit.runner.RunWith;
import cucumber.api.CucumberOptions;
import cucumber.api.junit.Cucumber;

@RunWith(Cucumber.class)
@CucumberOptions(features = "src/test/resources")
public class CucumberTest {}
```

Gherkin is a DSL language used to describe an application feature that needs to be tested. Here is our person Gherkin feature definition:

```gherkin
Feature: Persons can be retrieved  
  Scenario: client makes call to GET persons    
    When the client wants persons
    Then the client receives persons
```

The next step is to create a class with a web client to call our get persons end-point:

```java
package com.jos.dem.springboot.cucumber;

import com.jos.dem.springboot.cucumber.model.Person;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.web.reactive.function.client.WebClient;

import reactor.core.publisher.Flux;

@ContextConfiguration(classes = DemoApplication.class)
@WebAppConfiguration
public class SpringIntegrationTest {

  @Autowired
  private WebClient webClient;

  Flux<Person> executeGet() throws Exception {
    return webClient.get().uri("").retrieve()
    .bodyToFlux(Person.class);
  }
  
}
```

This `WebClient` is actually a bean defined in our `DemoApplication` as follow:

```java
package com.jos.dem.springboot.cucumber;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.reactive.function.client.WebClient;

@SpringBootApplication
public class DemoApplication {

  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }  

  @Bean
  WebClient getWebClient() {
    return WebClient.create("http://localhost:8080/persons");
  }

}
```

Now let's create the method in the Java class to correspond to this test case scenario:

```java
package com.jos.dem.springboot.cucumber;

import static org.junit.Assert.assertTrue;
import static org.junit.Assert.assertEquals;

import com.jos.dem.springboot.cucumber.model.Person;

import java.util.List;

import cucumber.api.java.en.Then;
import cucumber.api.java.en.When;
import reactor.core.publisher.Flux;

public class DefinitionIntegrationTest extends SpringIntegrationTest {
  private List<Person> persons;

  @When("^the client wants persons$")	
  public void shouldCallPersons() throws Exception {
    Flux<Person> result = executeGet();  
    persons = result.collectList().block();
  }

  @Then("^the client receives persons$")
  public void shouldGetPersons() throws Exception {      
    assertEquals(5 , persons.size());
    assertTrue(persons.contains(new Person("josdem", "joseluis.delacruz@gmail.com")));
    assertTrue(persons.contains(new Person("tgrip", "tgrip@email.com")));
    assertTrue(persons.contains(new Person("edzero", "edzero@email.com")));
    assertTrue(persons.contains(new Person("skuarch", "skuarch@email.com")));
    assertTrue(persons.contains(new Person("jeduan", "jeduan@email.com")));
  }

}
```

We can do a test run via Gradle task using command line as follow: 

```bash
gradle test
```

*Output:*

```bash
:test > Executing test com.jos.dem.springboot.cucumber.CucumberTest
BUILD SUCCESSFUL in 19s
5 actionable tasks: 2 executed, 3 up-to-date
```

To download the project:

```bash
git clone https://github.com/josdem/spring_boot_cucumber.git
```

To run the project:

```bash
gradle bootRun
```


[Return to the main article](/techtalk/spring)
