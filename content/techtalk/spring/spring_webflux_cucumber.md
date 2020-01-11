+++
title =  "Spring Webflux Cucumber"
categories = ["techtalk", "code","spring boot"]
tags = ["josdem", "techtalks","programming","technology","Java", "Cucumber","spring boot"]
date = "2018-03-30T19:42:17-06:00"
description = "BDD (Behavior-driven development) is a technique very similar to implement UAT (User Acceptance Testing) in a software project. Usually is a good idea to use BDD to reprecent how users can define application behaviour, so in that way you can represent user stories in test scenarios aka. feature testing."
+++

BDD (Behavior-driven development) is a technique very similar to implement UAT (User Acceptance Testing) in a software project. Usually is a good idea to use BDD to represent how users can define application behaviour, so in that way you can represent user stories in test scenarios aka. feature testing. This time I am going to show you how integrate [Cucumber](https://cucumber.io/) to a Spring Webflux application, Cucumber is a very powerful testing framework written in the [Ruby programming language](https://en.wikipedia.org/wiki/Ruby_(programming_language)), which follows the BDD methodology. **NOTE:** If you need to know what tools you need to have installed in your computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring/spring_boot)


* [Spring Boot Cucumber with Gradle](#Gradle)
* [Spring Boot Cucumber with Maven](#Maven)
* [Spring Boot Cucumber with Tags](#Tags)
* [Spring Boot Cucumber Reports](#Reports)

<a name="Gradle">
## Using Gradle
</a>

Let's start creating a new Spring Boot project with Webflux and Lombok as dependencies:

```bash
spring init --dependencies=webflux,lombok --build=gradle --language=java spring-webflux-cucumber
```

Here is the complete `build.gradle` file generated:

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

Then add Cucumber dependencies

```groovy
testImplementation("info.cukes:cucumber-java:$cucumberVersion")
testImplementation("info.cukes:cucumber-junit:$cucumberVersion")
testImplementation("info.cukes:cucumber-spring:$cucumberVersion")
```

Now let's create a simple POJO retrieve information from an end point using Spring WebFlux.

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

Lombok is a great tool to avoid boilerplate code, for knowing more please go [here](https://projectlombok.org/). Next step is to create a person controller so we can define `GET/persons` and `GET/persons` by nickname endpoints. `@RestController` indicates that we don't want to render views, but write the results straight into the response body instead.`@GetMapping` tells Spring that this is the place to route persons calls, it is the shorthand annotation for `@RequestMapping(method=RequestMethod.GET)`.

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

In order to complete our base project let's create `PersonService` to bring our person data:

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

Here is our mplementation:

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

[Gherkin](https://en.wikipedia.org/wiki/Cucumber_(software)#Gherkin_language) is a DSL language used to describe an application feature that needs to be tested. Here is our person Gherkin feature definition file `src/test/resources/person.feature`:

```gherkin
Feature: We can retrieve person data
  Scenario: We can retrieve all persons
    When I request all persons
    Then I validate all persons
  Scenario: We can retrieve specific person
    When I request person by nickname "josdem"
    Then I validate person data
```

Now, let's create a class with a web client to create request to our persons end-point:

```java
package com.jos.dem.springboot.cucumber;

import reactor.core.publisher.Mono;
import reactor.core.publisher.Flux;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.web.reactive.function.client.WebClient;

import com.jos.dem.springboot.cucumber.model.Person;

@ContextConfiguration(classes = CucumberApplication.class)
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

This `WebClient` is a bean defined in our `CucumberApplication` as follow:

```java
package com.jos.dem.springboot.cucumber;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.reactive.function.client.WebClient;

@SpringBootApplication(exclude = MongoAutoConfiguration.class)
public class CucumberApplication {

  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }

  @Bean
  WebClient getWebClient() {
    return WebClient.create("http://localhost:8080/");
  }

}
```

`@SpringBootApplication` disables MongoDB database auto-configuration. It is time to execute this command, so we can get our Spring Boot application up and running.

```bash
gradle bootRun
```

Now let's create get persons test scenario

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

And here is our get person test scenario

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

We can test using Gradle with this command:

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

You can do the same using Maven, the only difference is that you need to specify `--build=maven` parameter in the spring init command line:

```bash
spring init --dependencies=webflux,lombok --build=maven --language=java spring-webflux-cucumber
```

This is the `pom.xml` file generated along with Cucumber dependency on it added manualy:

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

Now you can execute this command, so we can get our Spring Boot application up and running with Maven.

```bash
mvn spring-boot:run
```

Finally this command to run our test scenario:

```bash
mvn test
```

<a name="Tags">
## Cucumber Tags
</a>

If you got many different feature files which cover different functionality of the application and  you want to execute only a specific feature file, you can use Cucumber tags from command line like this:

```bash
gradle -Dcucumber.options="--tags @SmokeTest" test
```

And this would be our feature file with `@SmokeTest` annotation:

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

**Note:** You need to pass system properties variables from command line to the `gradle test` task in order to do that, please add this lines to the `build.gradle` file

```groovy
test {
  systemProperties = System.properties
}
```

In case you are using Maven, use this command:

```bash
mvn -Dcucumber.options="--tags @SmokeTest" test
```

<a name="Reports">
## Reports
</a>

We easily can use a plugin so that we will be able to integrate [Extent Reports](http://extentreports.com/) to our project, in order to do that you need to add this dependencies:

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

Now in our Cucumber test runner class add the `com.vimalselvam.cucumber.listener.ExtentCucumberFormatter:target/reports/report.html` as a plugin

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

`extent-config.xml` is a config file with important elements such as:

* Report Theme : `<theme>` : standard or dark
* Document Encoding : `<encoding>` : UFT-8
* Title of the Report : `<documentTitle>` : This will display on the Browser Tab
* Name of the Report : `<reportName>` : This will display at the top of the Report

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

Now, if you execute the test cases, you will generate an Extent report like this:

<img src="/img/techtalks/spring/extent.png">

**Cucumber Reports**

Another cool plugin is [Cucumber Reports](https://wiki.jenkins.io/display/JENKINS/Cucumber+Reports+Plugin) is very stylish and good looking report, besides it is really easy to explore features, scenarios and steps. In order to integrate it you need to add a simple line of code:

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

Then after install Cucumber Reports plugin in your Jenkins server just create a post build action with this configuration:

<img src="/img/techtalks/spring/cucumber_reports.png">

So each time when you execute your Jenkins job you will have this kind of Cucumber Report:

<img src="/img/techtalks/spring/cucumber_reports1.png">

To browse the project go [here](https://github.com/josdem/spring-webflux-cucumber), to download the project:

```bash
git clone git@github.com:josdem/spring-webflux-cucumber.git
```


[Return to the main article](/techtalk/spring#Spring_Boot)

