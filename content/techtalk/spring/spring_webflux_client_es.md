+++
title =  "Spring Webflux el lado del Cliente"
categories = ["techtalk", "code","spring webflux"]
tags = ["josdem", "techtalks","programming","technology", "Spring WebFlux", "Reactive Web", "Java","spring webflux"]
date = "2018-03-28T11:36:40-06:00"
description = "This time we will see the client side in a Spring Webflux project."
+++

En el previo post [Spring Boot Server](/techtalk/spring/spring_webflux_server) cubrimos el tema de como crear un servidor reactivo, esta vez tocal el tiempo de ver el lado del cliente. Vamos a empezar por crear un proyecto con Webflux y Lombok como dependencias.

```bash
spring init --dependencies=webflux,lombok --build=gradle --language=java client
```

Aquí está el `build.gradle` generado:

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

Ahora, vamos a crear un simple y plano Java objeto para obtener la información del servidor reactivo.

```java
package com.jos.dem.webflux.model;

import lombok.Data;

@Data
public class Person {

  private String nickname;
  private String email;

}
```

Lombok es una gran herramienta para ahorrarnos código, para conocer más por favor ve [aquí](https://projectlombok.org/). El siguiente paso, vamos usar `CommandLineRunner` para empezar nuestro flujo de trabajo. El `CommandLineRunner` es una interfaz call back en Spring Boot, cuando nuestra aplcación arranca llamará al método start y le pasará los argumentos usando el método interno `run()`.


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

`WebClient` definido como `@Bean` es un client no-bloqueante, reactivo y su función aquí es hacer peticiones HTTP a nuestro servidor reactivo el cual usa Netty por default. No olvides correr este cliente en un puerto diferente usando la siguiente especificación in nuestro archivo `application.properties`.

```properties
server.port=8081
```

Para correr el proyecto:

```bash
gradle bootRun
```

**Usando Maven**

Tú puedes hacer lo mismo usando Maven, la única diferencia es que necesitas específicar el parámtero `--build=maven` en el comando `spring-init`:

```bash
spring init --dependencies=webflux,lombok --build=maven --language=java client
```

Este es el `pom.xml` generado:

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

Para correr el proyecto con Maven:

```bash
mvn spring-boot:run
```

Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/reactive-webflux-workshop), para descargar el proyecto:

```bash
git clone https://github.com/josdem/reactive-webflux-workshop.git
cd client
```

[Regresar al artículo principal](/techtalk/spring#Spring_Boot_Reactive_ES)
