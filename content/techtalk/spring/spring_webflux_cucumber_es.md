+++
title =  "Spring Webflux y Cucumber"
categories = ["techtalk", "code","spring boot"]
tags = ["josdem", "techtalks","programming","technology","Java", "Cucumber","spring boot"]
date = "2018-03-30T19:42:17-06:00"
description = "BDD (Behavior-driven development) is a technique very similar to implement UAT (User Acceptance Testing) in a software project. Usually is a good idea to use BDD to reprecent how users can define application behaviour, so in that way you can represent user stories in test scenarios aka. feature testing."
+++

BDD (Behavior-driven development) es una técnica muy similar a implementar UAT (User Acceptance Testing) en un proyecto de software. Usualmente es una buena idea usar BDD para representar como usuarios pueden definir el comportamiento de una aplicación, de esta manera podemos representar historias de usuario en test cases aka. feaure testing. Esta vez voy a mostrarte como integrar [Cucumber](https://cucumber.io/) a una aplicación Spring Webflux, Cucumber es un poderoso framework de pruebas escrito en [Ruby programming language](https://en.wikipedia.org/wiki/Ruby_(programming_language)), el cual sigue la metodología BDD. **Nota:** Si quieres saber que herramientas necesitas tener instaladas en tu computadora para poder familiarizarte con Spring Boot por favor visita mi previo post: [Spring Boot](/techtalk/spring/spring_boot). Entonces ejecuta este comando desde la terminal.


* [Spring Boot Cucumber with Gradle](#Gradle)
* [Spring Boot Cucumber with Maven](#Maven)
* [Spring Boot Cucumber with Tags](#Tags)
* [Spring Boot Cucumber Reports](#Reports)

<a name="Gradle">
## Using Gradle
</a>

Vamos a empezar creando un nuevo proyecto de Spring Boot con Webflux y Lombok como dependencias:

```bash
spring init --dependencies=webflux,lombok --build=gradle --language=java spring-webflux-cucumber
```

Aquí está el `build.gradle` generado:

```groovy
plugins {
  id 'org.springframework.boot' version '2.2.2.RELEASE'
  id 'io.spring.dependency-management' version '1.0.8.RELEASE'
  id 'java'
}

group = 'com.jos.dem.springboot.cucumber'
version = '0.0.1-SNAPSHOT'
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

Después agrega estas dependencias:

```groovy
testImplementation("info.cukes:cucumber-java:$cucumberVersion")
testImplementation("info.cukes:cucumber-junit:$cucumberVersion")
testImplementation("info.cukes:cucumber-spring:$cucumberVersion")
```

Ahora vamos a crear un POJO para obtener la información de un endpoint usando Spring Webflux.

```java
package com.jos.dem.springboot.cucumber.model;

import lombok.AllArgsConstructor;
import lombok.NoArgsConstructor;
import lombok.Data;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class Person {

  private String nickname;
  private String email;

}
```
Lombok es una gran herramienta para ahorrarnos código, para saber más acerca de Lombok por favor ve [aquí](https://projectlombok.org/). El siguiente paso es crear nuestro controlador de personas, así podemos definir `GET/persons` y `GET/persons` por nickname. `@RestController` indica que no queremos renderear vistas, pero queremos escribir los resultados directo en el cuerpo de la respuesta.`@GetMapping` le indica a Spring que es el lugar para rutear las peticiones de personas, es la forma corta de la anotación `@RequestMapping(method=RequestMethod.GET)`.

```java
package com.jos.dem.springboot.cucumber.controller;

import reactor.core.publisher.Mono;
import reactor.core.publisher.Flux;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.springboot.cucumber.model.Person;
import com.jos.dem.springboot.cucumber.service.PersonService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@RestController
@RequestMapping("/persons")
public class PersonController {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Autowired
  private PersonService personService;

  @GetMapping("/")
  public Flux<Person> findAll(){
    log.info("Calling find persons");
    return personService.getAll();
  }

  @GetMapping("/{nickname}")
  public Mono<Person> findById(@PathVariable String nickname){
    log.info("Calling find person by nickname: " + nickname);
    return personService.getByNickname(nickname);
  }

}
```
Para poder completar nuestro proyecto base, vamos a crear `PersonService` para traer los datos de persona.

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

Aquí está la implementación:

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
Junit runner usa Junit framework para poder ejecutar los test usando Cucumber. Lo que necesitamos definir es una sóla clase con la anotación `@RunWith(Cucumber.class)` y definir `@CucumberOptions` donde nosotros vamos a especificar la localización de los archivos Gherkin también conocidos como los archivos feature.

```java
package com.jos.dem.springboot.cucumber;

import org.junit.runner.RunWith;
import cucumber.api.CucumberOptions;
import cucumber.api.junit.Cucumber;

@RunWith(Cucumber.class)
@CucumberOptions(features = "src/test/resources")
public class CucumberTest {}
```

[Gherkin](https://en.wikipedia.org/wiki/Cucumber_(software)#Gherkin_language) es un lenguaje DSL usado para describir una aplicación que necesita ser testeada. Aquí está nuestro archivo feature `src/test/resources/person.feature`:

```gherkin
Feature: We can retrieve person data
  Scenario: We can retrieve all persons
    When I request all persons
    Then I validate all persons
  Scenario: We can retrieve specific person
    When I request person by nickname "josdem"
    Then I validate person data
```

Ahora, vamos a crear una clase con Webclient para poder hacer las peticiones al end-point `persons`:

```java
package com.jos.dem.springboot.cucumber;

import reactor.core.publisher.Mono;
import reactor.core.publisher.Flux;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.web.reactive.function.client.WebClient;

import com.jos.dem.springboot.cucumber.model.Person;

@ContextConfiguration(classes = DemoApplication.class)
@WebAppConfiguration
public class PersonIntegrationTest {

  @Autowired
  private WebClient webClient;

  Flux<Person> getPersons() throws Exception {
    return webClient.get()
      .uri("/persons/")
      .retrieve()
    .bodyToFlux(Person.class);
  }

  Mono<Person> getPerson(String nickname) throws Exception {
    return webClient.get()
      .uri("/persons/" + nickname)
      .retrieve()
    .bodyToMono(Person.class);
  }

}
```

Este es el `WebClient` definido en nuestro `CucumberApplication`:

```java
package com.jos.dem.springboot.cucumber;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.reactive.function.client.WebClient;

@SpringBootApplication
public class CucumberApplication {

  public static void main(String[] args) {
    SpringApplication.run(CucumberApplication.class, args);
  }

  @Bean
  WebClient webClient() {
    return WebClient.create("http://localhost:8080/");
  }

}
```

Es tiempo de ejecutar este comando, así podemos tener nuestra aplicación Spring Boot correndo y lista.

```bash
gradle bootRun
```

Ahora vamos a crear la implementación de prueba para obtener personas.

```java
package com.jos.dem.springboot.cucumber;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.Date;
import java.util.List;

import reactor.core.publisher.Flux;

import com.jos.dem.springboot.cucumber.model.Person;

import cucumber.api.java.After;
import cucumber.api.java.Before;
import cucumber.api.java.en.When;
import cucumber.api.java.en.Then;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class GetPersonsTest extends PersonIntegrationTest {

  private List<Person> persons;
  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Before
  public void setup() {
    log.info("Before any test execution");
  }

  @When("I request all persons")
  public void shouldGetPersons() throws Exception {
    log.info("Running: I request all persons at " + new Date());
    persons = getPersons().collectList().block();
  }

  @Then("I validate all persons")
  public void shouldValidatePersons() throws Exception {
    log.info("Running: I validate all persons at " + new Date());

    assertEquals(5 , persons.size());
    assertAll("person",
      () -> assertTrue(persons.contains(new Person("josdem", "joseluis.delacruz@gmail.com"))),
      () -> assertTrue(persons.contains(new Person("tgrip", "tgrip@email.com"))),
      () -> assertTrue(persons.contains(new Person("edzero", "edzero@email.com"))),
      () -> assertTrue(persons.contains(new Person("skuarch", "skuarch@email.com"))),
      () -> assertTrue(persons.contains(new Person("jeduan", "jeduan@email.com")))
    );
  }

  @After
  public void tearDown() {
    log.info("After all test execution");
  }

}
```

Y aquí está el escenario de prueba para obtener una persona en partícular

```java
package com.jos.dem.springboot.cucumber;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.Date;

import reactor.core.publisher.Flux;

import com.jos.dem.springboot.cucumber.model.Person;

import cucumber.api.java.After;
import cucumber.api.java.Before;
import cucumber.api.java.en.When;
import cucumber.api.java.en.Then;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class GetPersonTest extends PersonIntegrationTest {

  private Person person;
  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Before
  public void setup() {
    log.info("Before any test execution");
  }

  @When("^I request person by nickname \"([^\"]*)\"$")
  public void shouldGetPersonByNickname(String nickname) throws Exception {
    log.info("Running: I request person by nickname at " + new Date());
    person = getPerson(nickname).block();
  }

  @Then("I validate person data")
  public void shouldValidatePersonData() throws Exception {
    log.info("Running: I validate person data at " + new Date());

    assertAll("person",
      () -> assertEquals("josdem", person.getNickname()),
      () -> assertEquals("joseluis.delacruz@gmail.com", person.getEmail())
    );
  }

  @After
  public void tearDown() {
    log.info("After all test execution");
  }

}
```

Podemos ejecutar los test usando Gradle con este comando:

```bash
gradle test
```

*Output:*

```bash
> Task :test
BUILD SUCCESSFUL in 5s
5 actionable tasks: 5 executed
```

<a name="Maven">
## Using Maven
</a>

Tú puedes hacer lo mismo usando Maven, la única diferencia es que tienes que específicar el parámetro `--build=maven` en el comando `spring init`:

```bash
spring init --dependencies=webflux,lombok --build=maven --language=java spring-webflux-cucumber
```

Este es el `pom.xml` generado:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.2.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.jos.dem.springboot</groupId>
	<artifactId>cucumber</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>spring-webflux-cucumber</name>
	<description>Demo project for Spring Boot</description>

	<properties>
    <cucumber.version>1.2.6</cucumber.version>
		<java.version>12</java.version>
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
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<dependency>
			<groupId>io.projectreactor</groupId>
			<artifactId>reactor-test</artifactId>
			<scope>test</scope>
		</dependency>
    <dependency>
      <groupId>info.cukes</groupId>
      <artifactId>cucumber-java</artifactId>
      <version>${cucumber.version}</version>
    </dependency>
    <dependency>
      <groupId>info.cukes</groupId>
      <artifactId>cucumber-junit</artifactId>
      <version>${cucumber.version}</version>
    </dependency>
    <dependency>
      <groupId>info.cukes</groupId>
      <artifactId>cucumber-spring</artifactId>
      <version>${cucumber.version}</version>
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

Entonces puedes correr el proyecto usando este comando:

```bash
mvn spring-boot:run
```

Finalmente este comando para correr los tests:

```bash
mvn test
```

<a name="Tags">
## Cucumber Tags
</a>

Si tienes diferentes archivos feature que cubren diferente funcioncalidad de la aplicación y quieres sólo ejecutar cierto feature, puedes usar los tags de Cucumber desde la línea de comandos como sigue:

```bash
gradle -Dcucumber.options="--tags @SmokeTest" test
```

Y este sería el archivo feature con la anotación `@SmokeTest`:

```gherkin
@SmokeTest
Feature: We can retrieve person data
  Scenario: We can retrieve all persons
    When I request all persons
    Then I validate all persons
  Scenario: We can retrieve specific person
    When I request person by nickname "josdem"
    Then I validate person data
```

**Nota:** Tú necesitas pasar las variables de sistema desde la línea de comandos a la tarea test de Gradle y para poder hacer eso, por favor agrega estas líneas al archivo `build.gradle`

```groovy
test {
  systemProperties = System.properties
}
```

En caso que uses Maven, es este comando:

```bash
mvn -Dcucumber.options="--tags @SmokeTest" test
```

<a name="Reports">
## Reports
</a>

Fácilmente podemos usar un plugin que nos permita integrar [Extent Reports](http://extentreports.com/) a nuestro proyecto, para poder hacer eso necesitarás agregar estas dependencias:

**Gradle**

```groovy
testImplementation('com.aventstack:extentreports:3.1.1')
testImplementation('com.vimalselvam:cucumber-extentsreport:3.1.1')
```

**Maven**

```xml
<dependency>
  <groupId>com.vimalselvam</groupId>
  <artifactId>cucumber-extentsreport</artifactId>
  <version>3.1.1</version>
</dependency>
<dependency>
  <groupId>com.aventstack</groupId>
  <artifactId>extentreports</artifactId>
  <version>3.1.1</version>
</dependency>
```

Ahora en nuestra clase Cucumber agrega el nuevo `com.vimalselvam.cucumber.listener.ExtentCucumberFormatter:target/reports/report.html` como un plugin

```java
package com.jos.dem.springboot.cucumber;

import java.io.File;

import com.vimalselvam.cucumber.listener.Reporter;

import org.junit.runner.RunWith;
import org.junit.AfterClass;

import cucumber.api.CucumberOptions;
import cucumber.api.junit.Cucumber;

@RunWith(Cucumber.class)
@CucumberOptions(features = "src/test/resources",
                 format = "pretty",
                 plugin = "com.vimalselvam.cucumber.listener.ExtentCucumberFormatter:target/reports/report.html")
public class CucumberTest {

  @AfterClass
  public static void teardown() {
    Reporter.loadXMLConfig(new File("src/test/resources/config/extent-config.xml"));
  }

}
```

`extent-config.xml` es un archivo de configuración con algunos elementos imporantes como:

* Report Theme : `<theme>` : standard es oscuro
* Document Encoding : `<encoding>` : UFT-8
* Title of the Report : `<documentTitle>` : Es el título que se mostrará en el Tab del browser
* Name of the Report : `<reportName>` : Este título se mostrará en el inicio del reporte

```xml
<?xml version="1.0" encoding="UTF-8"?>
<extentreports>
  <configuration>
    <theme>standard</theme>
    <encoding>UTF-8</encoding>
    <protocol>https</protocol>
    <documentTitle>Extent</documentTitle>
    <reportName>Spring Boot - Cucumber Report</reportName>
    <scripts>
      <![CDATA[
        $(document).ready(function() {

        });
      ]]>
    </scripts>
    <styles>
      <![CDATA[

      ]]>
    </styles>
  </configuration>
</extentreports>
```

Ahora, si ejecutamos los test cases, podrás genera el reporte Extent como sigue:

<img src="/img/techtalks/spring/extent.png">

**Reportes Cucumber**

Otro buen plugin es [Cucumber Reports](https://wiki.jenkins.io/display/JENKINS/Cucumber+Reports+Plugin) es muy estilizado y atractivo reporte, además facilita explorar los features, escenarios y pasos. Para poder generarlo necesitas una simple línea de código:

```java
package com.jos.dem.jugoterapia.cucumber;

import java.io.File;

import com.vimalselvam.cucumber.listener.Reporter;

import org.junit.runner.RunWith;
import org.junit.AfterClass;

import cucumber.api.CucumberOptions;
import cucumber.api.junit.Cucumber;

@RunWith(Cucumber.class)
@CucumberOptions(features = "src/test/resources",
                 format = {"pretty","json:target/reports/cucumber.json"},
                 plugin = "com.vimalselvam.cucumber.listener.ExtentCucumberFormatter:build/reports/report.html")
public class CucumberTest {

  @AfterClass
  public static void teardown() {
    Reporter.loadXMLConfig(new File("src/test/resources/config/extent-config.xml"));
  }
}
```

Después de instalar el plugín de reportes de Cucumber en tu Jenkins server sólo crea un post build action con esta configuración.

<img src="/img/techtalks/spring/cucumber_reports.png">

Así cada vez que ejecutes un Jenkins job encontrarás este tipo de reporte de Cucumber:

<img src="/img/techtalks/spring/cucumber_reports1.png">

Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/spring-webflux-cucumber), para descargar el proyecto:

```bash
git clone git@github.com:josdem/spring-webflux-cucumber.git
```


[Regresar al artículo principal](/techtalk/spring#Spring_Boot_Reactive_ES)


