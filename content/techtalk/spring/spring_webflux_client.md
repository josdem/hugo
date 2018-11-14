+++
title =  "Spring Webflux Client"
categories = ["techtalk", "code","spring webflux"]
tags = ["josdem", "techtalks","programming","technology", "Spring WebFlux", "Reactive Web", "Java","spring webflux"]
date = "2018-03-28T11:36:40-06:00"
description = "This time we will see the client side in a Spring Webflux project."
+++

In previous [Spring Boot Server](/techtalk/spring/spring_webflux_server) we covered web reactive server, this time we will see the client side. Let's start creating a new project with Webflux, Mongo Reactive and Lombok as dependencies:

```bash
spring init --dependencies=webflux,data-mongodb-reactive,lombok --build=gradle --language=java client
```

Here is the complete `build.gradle` file generated:

```groovy
buildscript {
  ext {
    springBootVersion = '2.1.0.RELEASE'
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
  implementation('org.springframework.boot:spring-boot-starter-webflux')
  implementation('org.springframework.boot:spring-boot-starter-data-mongodb-reactive')
  compileOnly('org.projectlombok:lombok')
  testImplementation('org.springframework.boot:spring-boot-starter-test')
  testImplementation('io.projectreactor:reactor-test')
}
```

Now let's create a simple POJO to retrieve information from our reactive server.

```java
package com.jos.dem.webflux.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

@Data
@Document
@NoArgsConstructor
@AllArgsConstructor
public class Person {

  @Id
  private String uuid;
  private String nickname;
  private String email;

}
```

Lombok is a great tool to avoid boilerplate code, for knowing more please go [here](https://projectlombok.org/). Next, we are going to use `CommandLineRunner` to start our workflow. The `CommandLineRunner` is a call back interface in Spring Boot, when Spring Boot starts will call it and pass in args through a `run()` internal method.

```java
package com.jos.dem.webflux;

import org.springframework.context.annotation.Bean;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.reactive.function.client.WebClient;

import com.jos.dem.webflux.model.Person;

@SpringBootApplication
public class PersonApplication {

  public static void main(String[] args) {
    SpringApplication.run(PersonApplication.class, args);
  }

  @Bean
  WebClient webClient() {
    return WebClient.create("http://localhost:8080/persons");
  }

  @Bean
  CommandLineRunner run(WebClient client){
    return args -> {
      client.get().uri("").retrieve()
        .bodyToFlux(Person.class)
        .subscribe(System.out::println);
    };
  }

}
```

`WebClient` defined as `@Bean` is a non-blocking, reactive client for performing HTTP requests with our reactive web server, and again Netty is used by default. Do not forget to run this client in different port using the following specification in our `application.properties` file

```properties
server.port=8081
```

To run the project:

```bash
gradle bootRun
```

To run the project with Maven:

```bash
mvn spring-boot:run
```

To browse the project go [here](https://github.com/josdem/reactive-webflux-workshop), to download the project:

```bash
git clone https://github.com/josdem/reactive-webflux-workshop.git
git fetch
git checkout feature/client
```


[Return to the main article](/techtalk/spring#Spring_Boot)
