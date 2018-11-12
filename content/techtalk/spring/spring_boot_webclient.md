+++
title =  "Spring Boot WebClient"
description = "WebClient is a reactive client that provides an alternative to RestTemplate. It exposes a functional, fluent API and relies on non-blocking I/O which allows it to support high concurrency more efficiently than the RestTemplate. WebClient is a natural fit for streaming scenarios and depends on a lower level HTTP client library to execute requests and that support is pluggable"
date = "2018-06-13T15:54:27-04:00"
tags = ["josdem", "techtalks","programming","technology", "Spring Boot WebClient", "WebFlux WebClient"]
categories = ["techtalk", "code", "WebFlux"]
+++

[WebClient](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-webclient.html) is a reactive client that provides an alternative to RestTemplate. It exposes a functional, fluent API and relies on non-blocking I/O which allows it to support high concurrency more efficiently than the RestTemplate. WebClient is a natural fit for streaming scenarios and depends on a lower level HTTP client library to execute requests and that support is pluggable. WebClient uses the same codecs as WebFlux server applications do, and shares a common base package, some common APIs, and infrastructure with the server functional web framework. The API exposes Reactor Flux and Mono types. By default it uses Reactor Netty as the HTTP client library but others can be plugged in through a custom ClientHttpConnector.

By comparison to the RestTemplate, the WebClient is:

* Non-blocking, reactive, and supports higher concurrency with less hardware resources.
* Provides a functional API that takes advantage of Java 8 lambdas.
* Supports both synchronous and asynchronous scenarios.
* Supports streaming up or down from a server.

The RestTemplate is not a good fit for use in non-blocking applications, and therefore Spring WebFlux application should always use the WebClient. The WebClient should also be preferred in Spring MVC, in most high concurrency scenarios, and for composing a sequence of remote, inter-dependent calls. Let's start creating a new project using this command:


**Using Gradle**

```bash
spring init --dependencies=webflux,lombok --language=java --build=gradle spring-boot-webclient
```

Here is the `build.gradle` file generated:

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

group = 'com.jos.dem.springboot.webclient'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
  mavenCentral()
}

dependencies {
  compile('org.springframework.boot:spring-boot-starter-webflux')
  compile('org.springframework.boot:spring-boot-starter')
  compile('org.projectlombok:lombok')
  testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

**NOTE:** If you need to know what tools you need to have installed in your computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot). Next step is to create a service with WebClient so we can send request and get responses in our project.

## Retrieve

The `retrieve()` method is the easiest way to get a response body and decode it:

```java
package com.jos.dem.springboot.webclient.service.impl;

import static org.springframework.http.MediaType.APPLICATION_JSON;

import reactor.core.publisher.Mono;
import org.springframework.stereotype.Service;
import org.springframework.web.reactive.function.client.WebClient;

import com.jos.dem.springboot.webclient.model.Beverage;
import com.jos.dem.springboot.webclient.service.BeverageService;

@Service
public class BeverageServiceImpl implements BeverageService {

  private WebClient client = WebClient.create("https://jugoterapia.josdem.io/jugoterapia-server");

  public Mono<Beverage> getBeverage(Long id){
    return client.get()
      .uri("/beverages/{id}", id).accept(APPLICATION_JSON)
      .retrieve()
      .bodyToMono(Beverage.class);
  }

}
```

In this example we are going to consume a `RESTFul` service within this project [Jugoterapia](https://github.com/josdem/jugoterapia-spring-boot) Which is an Android application mainly focused in improve your healty based in juice recipes, this project is the server side, it is exposing recipes and beverages as API service.

Beverage service interface:

```java
package com.jos.dem.springboot.webclient.service;

import reactor.core.publisher.Mono;
import com.jos.dem.springboot.webclient.model.Beverage;

public interface BeverageService {

  Mono<Beverage> getBeverage(Long id);

}
```

Now let's create a simple POJO to retrieve information from our API.

```java
package com.jos.dem.springboot.webclient.model;

import lombok.AllArgsConstructor;
import lombok.NoArgsConstructor;
import lombok.Data;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Beverage {

  private Long id;
  private String name;
  private String ingredients;
  private String recipe;

}
```

Lombok is a great tool to avoid boilerplate code, for knowing more please go [here](https://projectlombok.org/). Next, we are going to use `CommandLineRunner` to start our workflow. The `CommandLineRunner` is a call back interface in Spring Boot, when Spring Boot starts will call it and pass in args through a `run()` internal method.

```java
package com.jos.dem.springboot.webclient;

import org.springframework.context.annotation.Bean;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import com.jos.dem.springboot.webclient.service.BeverageService;

@SpringBootApplication
public class DemoApplication {

  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }

  @Bean
  CommandLineRunner run(BeverageService beverageService){
    return args -> {
      beverageService.getBeverage(35)
      .subscribe(System.out::println);
    };
  }

}
```

Now if you run the project:

```bash
gradle bootRun
```

You will see a request to the API service and a Json response:

```bash
2018-06-15 22:55:38.550  INFO 1454 --- [NettyWebServer  : Netty started on port(s): 8080
2018-06-15 22:55:38.554  INFO 1454 --- [DemoApplication : Started DemoApplication
Beverage(id=35, name=Jugo nutritivo (Zanahoria), ingredients=4 Zanahorias,1 Tallo de apío,1 Pera,5 hojas de espinacas, recipe=Lava perfectamente todos los ingrendientes. Pasa la zanahoria por el extractor, el apio, las espinacas y la pera. Mezcla todo perfectamente y bebe de inmediato. La espinaca es una excelente fuente de hierro. Promueve el transporte y depósito de oxí­geno en los tejidos, aumenta la fuerza muscular, ayuda a bajar de peso, favorece el tránsito intestinal, beneficia a mujeres embarazadas y niños debido a su contenido de ácido fólico (vitamina B9), mejora la visión y mantiene la presión arterial balanceada.)
```

**Using Maven**

You can do the same using Maven, the only difference is that you need to specify `--build=maven` parameter in the `spring init` command line:

```bash
spring init --dependencies=webflux,lombok --language=maven --build=gradle spring-boot-webclient
```

And when you run your project use this command:

```bash
mvn spring-boot:run
```

This is the `pom.xml` file generated:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem.springboot</groupId>
  <artifactId>weblcient</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-boot-webclient</name>
  <description>Shows how to use reactive client WebClient in a Spring Boot application</description>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.6.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>io.projectreactor</groupId>
      <artifactId>reactor-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>

</project>
```

To browse the project go [here](https://josdem.io/techtalk/spring/spring_boot_webclient/), to download the project:

```bash
git clone https://github.com/josdem/spring-boot-webclient.git
```

[Return to the main article](/techtalk/spring#Spring_Boot)
