+++
title =  "Spring Webflux Client"
categories = ["techtalk", "code","spring webflux"]
tags = ["josdem", "techtalks","programming","technology", "Spring WebFlux", "Reactive Web", "Java","spring webflux"]
date = "2018-03-28T11:36:40-06:00"
description = "This time we will see the client side in a Spring Webflux project."
+++

In previous [Spring Boot Server](/techtalk/spring/spring_webflux_server) we covered web reactive server, this time we will see the client side. Let's start creating a new project with Webflux and Lombok as dependencies:

```bash
spring init --dependencies=webflux,lombok --build=gradle --language=java client
```

Here is the complete `build.gradle` file generated:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.5.RELEASE'
  id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.webflux'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 11

configurations {
  compileOnly {
    extendsFrom annotationProcessor
  }
}

repositories {
  mavenCentral()
}

dependencies {
  implementation('org.springframework.boot:spring-boot-starter-webflux')
  compileOnly('org.projectlombok:lombok')
  annotationProcessor 'org.projectlombok:lombok'
  testImplementation('org.springframework.boot:spring-boot-starter-test')
  testImplementation('io.projectreactor:reactor-test')
}
```

Now let's create a simple POJO to retrieve information from our reactive server.

```java
package com.jos.dem.webflux.model;

import lombok.Data;

@Data
public class Person {

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

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@SpringBootApplication
public class PersonApplication {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  public static void main(String[] args) {
    SpringApplication.run(PersonApplication.class, args);
  }

  @Bean
  WebClient webClient() {
    return WebClient.create("http://localhost:8080");
  }

  @Bean
  CommandLineRunner run(WebClient client){
    return args -> {
      client.get().uri("/persons").retrieve()
        .bodyToFlux(Person.class)
        .subscribe(person -> log.info("person: {}", person));
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

**Using Maven**

You can do the same using Maven, the only difference is that you need to specify `--build=maven` parameter in the `spring init` command line:

```bash
spring init --dependencies=webflux,lombok --build=maven --language=java client
```

This is the pom.xml file generated:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem.webflux</groupId>
  <artifactId>reactive-webflux-workshop</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>demo</name>
  <description>Demo project for Spring Webflux</description>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.5.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>11</java.version>
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

To run the project with Maven:

```bash
mvn spring-boot:run
```

To browse the project go [here](https://github.com/josdem/reactive-webflux-workshop), to download the project:

```bash
git clone https://github.com/josdem/reactive-webflux-workshop.git
cd client
```

[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
