+++
title = "Getting Started with Spring Webflux"
categories = ["techtalk", "code","spring webflux"]
tags = ["josdem", "techtalks","programming","technology", "Webflux", "Reactive Webflux","spring webflux"]
date = "2018-03-25T12:57:29-06:00"
description = "Spring Boot now embraces reactive programming which is a non-blocking asynchronous applications and event-driven. Spring Framework uses Reactor internally for its own reactive support. Reactor is a Reactive Streams implementation that further extends the basic Publisher contract with the Flux and Mono."
+++

Spring Boot now embraces reactive programming which is a non-blocking asynchronous applications and event-driven. Spring Framework uses Reactor internally for its own reactive support. Reactor is a Reactive Streams implementation that further extends the basic Publisher contract with the Flux and Mono.

Let's start creating a new Spring Boot project with Webflux, Mongo Reactive and Lombok as dependencies:

```bash
spring init --dependencies=webflux,data-mongodb-reactive,lombok --build=gradle --language=java reactive-webflux-workshop
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

group = 'com.jos.dem.webflux'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
  mavenCentral()
}

dependencies {
  compile('org.springframework.boot:spring-boot-starter-webflux')
  compile('org.codehaus.groovy:groovy')
  compile('org.projectlombok:lombok')
  compile('org.springframework.boot:spring-boot-starter-data-mongodb-reactive')
  testCompile('org.springframework.boot:spring-boot-starter-test')
  testCompile('io.projectreactor:reactor-test')
}
```

Now let's create a simple POJO to store and retrieve information from our MongoDB.

```java
package com.jos.dem.webflux.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

import lombok.AllArgsConstructor;
import lombok.NoArgsConstructor;
import lombok.ToString;
import lombok.Data;

@Document
@AllArgsConstructor
@ToString
@NoArgsConstructor
@Data
public class Person {

  @Id
  private String uuid;
  private String nickname;
  private String email;

}
```

Lombok is a great tool to avoid boilerplate code, for knowing more please go [here](https://projectlombok.org/)

Spring Data now is supporting a full reactive experience with MongoDB, Couchbase, Redis and Casandra, in this case let's create a `PersonRepository` using `ReactiveMongoRepository` as we going to be reactive.

```java
package com.jos.dem.webflux.repository;

import org.springframework.data.mongodb.repository.ReactiveMongoRepository;
import reactor.core.publisher.Flux;
import com.jos.dem.webflux.model.Person;

public interface PersonRepository extends ReactiveMongoRepository<Person, String> {}
```

Next, we are going to use `CommandLineRunner` to start our workflow. The `CommandLineRunner` is a call back interface in Spring Boot, when Spring Boot starts will call it and pass in args through a `run()` internal method.

```java
package com.jos.dem.webflux;

import org.springframework.context.annotation.Bean;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class PersonApplication {

  public static void main(String[] args) {
    SpringApplication.run(PersonApplication.class, args);
  }

  @Bean
  CommandLineRunner start(){
    return args -> {
      System.out.println("Hello World!");
    };
  }

}
```

It is time to create a list of persons and store them in our MongoDB

```java
package com.jos.dem.webflux;

import org.springframework.context.annotation.Bean;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import reactor.core.publisher.Flux;

import java.util.UUID;
import java.util.stream.Stream;

import com.jos.dem.webflux.model.Person;
import com.jos.dem.webflux.repository.PersonRepository;

@SpringBootApplication
public class PersonApplication {

  public static void main(String[] args) {
    SpringApplication.run(PersonApplication.class, args);
  }

  @Bean
  CommandLineRunner start(PersonRepository personRepository){
    return args -> {
      Stream.of("josdem", "tgrip", "edzero", "skuarch", "siedrix")
      .map(nickname -> new Person(UUID.randomUUID().toString(), nickname, nickname + "@email.com"))
      .forEach(person -> personRepository.save(person).subscribe());
    };
  }

}
```

**IMPORTANT:** The logic implemented in the operators is only executed when data starts to flow, and that does not happen until you use `subscribe()` method.

That's it, now we are storing `Person` objects to our MongoDB, let's add some code to clean our database, insert and show persons.

```java
package com.jos.dem.webflux;

import org.springframework.context.annotation.Bean;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import reactor.core.publisher.Flux;

import java.util.UUID;
import java.util.stream.Stream;

import com.jos.dem.webflux.model.Person;
import com.jos.dem.webflux.repository.PersonRepository;

@SpringBootApplication
public class PersonApplication {

  public static void main(String[] args) {
    SpringApplication.run(PersonApplication.class, args);
  }

  @Bean
  CommandLineRunner start(PersonRepository personRepository){
    return args -> {
      personRepository.deleteAll().subscribe();

      Stream.of("josdem", "tgrip", "edzero", "skuarch", "siedrix")
      .map(nickname -> new Person(UUID.randomUUID().toString(), nickname, nickname + "@email.com"))
      .forEach(person -> personRepository.save(person).subscribe());

      personRepository.findAll().log().subscribe(System.out::println);
    };
  }

}
```

Do not forget to add your MongoDB credentials information to your `application.properties` file:

```properties
spring.data.mongodb.database=reactive_webflux
spring.data.mongodb.host=localhost
spring.data.mongodb.username=username
spring.data.mongodb.password=password
```

To browse the project go [here](https://github.com/josdem/reactive-webflux-workshop), to download the project:

```bash
git clone https://github.com/josdem/reactive-webflux-workshop.git
git fetch
git checkout feature/setup
```

To run the project:

```bash
gradle bootRun
```


[Return to the main article](/techtalk/spring#Spring_Boot)
