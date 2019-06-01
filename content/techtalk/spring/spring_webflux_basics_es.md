+++
title = "Empezando con Spring Webflux"
categories = ["techtalk", "code","spring webflux"]
tags = ["josdem", "techtalks","programming","technology", "Webflux", "Reactive Webflux","spring webflux"]
date = "2018-03-25T12:57:29-06:00"
description = "Spring Boot now embraces reactive programming which is a non-blocking asynchronous applications and event-driven. Spring Framework uses Reactor internally for its own reactive support. Reactor is a Reactive Streams implementation that further extends the basic Publisher contract with the Flux and Mono."
+++

Spring Boot ahora fomenta al programación reactiva la cual es tecnología no-bloqueante, asíncrona y dirigida por eventos. Spring Framework usa Reactor internamente para su suporte reactivo. Reactor is un proyecto basado en la implementación de streams usando el patrón de diseño Publisher/Subscriber, los publishers en Webflux se llaman Flux y Mono. **NOTA:** Si quieres saber que necesitas tener instalado en tu computadora para crear fácilmente un proyecto de Spring boot, por favor visita mi previo post: [Spring Boot](/techtalk/spring_boot). Vamos a empezar creando un  nuevo proyecto de Spring Boot con Webflux, Mongo Reactive y Lombok como dependencias:

```bash
spring init --dependencies=webflux,data-mongodb-reactive,lombok --build=gradle --language=java reactive-webflux-workshop
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
  compileOnly 'org.projectlombok:lombok'
  annotationProcessor 'org.projectlombok:lombok'
  implementation('org.springframework.boot:spring-boot-starter-data-mongodb-reactive')
  testImplementation('org.springframework.boot:spring-boot-starter-test')
  testImplementation('io.projectreactor:reactor-test')
}
```

Ahora es momento de crear un simple plano objeto de Java el cual nos ayudará a guardar y obtener información de la base de datos MongoDB.

```java
package com.jos.dem.webflux.model;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

import lombok.Data;
import lombok.AllArgsConstructor;

@Data
@Document
@AllArgsConstructor
public class Person {

  @Id
  private String nickname;
  private String email;

}
```

La anotación `@Data` genera setters y getters, el método `toString()`, `equals()`, `hashcode()` y un constructor por cada atributo de la clase. Lombok es una gran herramienta para ahorrarnos código, para saber más acerca de Lombok por favor ve [aquí](https://projectlombok.org/). Spring Data ahora soporta la experiencia reactiva con MongoDB, Couchbase, Redis y Casandra, en este caso usaremos `PersonRepository` que extiende `ReactiveMongoRepository`.

```java
package com.jos.dem.webflux.repository;

import org.springframework.data.mongodb.repository.ReactiveMongoRepository;

import com.jos.dem.webflux.model.Person;

public interface PersonRepository extends ReactiveMongoRepository<Person, String> {}
```

Ahora, vamos a usar `CommandLineRunner` para iniciar nuestro flujo de trabajo. El `CommandLineRunner` es una interfaz call back en Spring Boot, cuando nuestra aplicación arranca llamará al método start y le pasará los argumentos unsando el método interno `run()`

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

Es tiempo de crear una lista de personas y guardarlas en nuestra base de datos MongoDB.

```java
package com.jos.dem.webflux;

import org.springframework.context.annotation.Bean;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import reactor.core.publisher.Flux;

import java.util.stream.Stream;

import com.jos.dem.webflux.model.Person;
import com.jos.dem.webflux.repository.PersonRepository;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@SpringBootApplication
public class PersonApplication {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  public static void main(String[] args) {
    SpringApplication.run(PersonApplication.class, args);
  }

  @Bean
  CommandLineRunner start(PersonRepository personRepository){
    return args -> {

      Flux.just(
          new Person("josdem", "joseluis.delacruz@gmail.com"),
          new Person("tgrip", "tgrip@email.com"),
          new Person("edzero", "edzero@email.com"),
          new Person("siedrix", "siedrix@email.com"),
          new Person("mkheck", "mkheck@email.com"))
        .flatMap(personRepository::save)
        .subscribe(person -> log.info("person: {}", person));

    };
  }

}
```

La lógica implementada en los operadores es sólamente ejecutada cuando nuestros datos empiezan a fluir, y la magia pasa cuando usamos el método `subscribe()`. Así es, ahora estamos guardando `Person` en nuestra base de datos MongoDB. Para poder ejecutar este ejemplo necesitas crear una base de datos en MongoDB con autorización habilitada para un usuario, no olvides agregar esas credenciales en tu archivo `application.properties`.

```properties
spring.data.mongodb.database=reactive_webflux
spring.data.mongodb.host=localhost
spring.data.mongodb.username=username
spring.data.mongodb.password=password
```

Para ejecutar el proyecto.

```bash
gradle bootRun
```

**Using Maven**

Tú puedes hacer lo mismo usando Maven, la única diferencia es que tienes que específicar el parámetro `--build=maven` en el comando `spring init`:

```bash
spring init --dependencies=webflux,data-mongodb-reactive,lombok --build=maven --language=java reactive-webflux-workshop
```

Entonces puedes correr el proyecto usando este comando:

```bash
mvn spring-boot:run
```

Este es el `pom.xml` generado:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>reactive-webflux-workshop</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>demo</name>
	<description>Demo project for Spring Boot</description>

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
			<artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
		</dependency>
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

Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/reactive-webflux-workshop), para descargar el proyecto:

```bash
git clone https://github.com/josdem/reactive-webflux-workshop.git
cd server
```

[Regresar al artículo principal](/techtalk/spring#Spring_Boot_Reactive_ES)
