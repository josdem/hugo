+++
title =  "Webclient Cucumber and Junit 5"
description = "Webflux WebClient along with Cucumber and Junit 5 to create client test automation to the GitHub API v3"
date = "2018-07-09T20:31:06-04:00"
tags = ["josdem", "techtalks","programming","technology","spring boot", "webclient", "cucumber", "junit5"]
categories = ["techtalk", "code", "spring boot", "cucumber", "junit5"]
+++

This time I will show you how to combine Webflux [WebClient](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-webclient.html) along with [Cucumber](https://cucumber.io/) and [Junit 5](https://junit.org/junit5/) in order to consume [GitHub API v3](https://developer.github.com/v3/?) public REST API. First, let’s start creating a new Spring Boot project with Webflux and Lombok as dependencies:

```bash
spring init --dependencies=webflux,lombok --build=gradle --language=java spring-boot-web-client
```

Here is the complete `build.gradle` file generated:

```groovy
buildscript {
	ext {
		springBootVersion = '2.0.4.RELEASE'
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

**NOTE:** If you want to know what tools you need to have installed in your computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot)

Now add latest Cucumber and Junit 5 Framework dependencies to your `build.gradle` file:

```groovy
testCompile("info.cukes:cucumber-java:$cucumberVersion")
testCompile("info.cukes:cucumber-junit:$cucumberVersion")
testCompile("info.cukes:cucumber-spring:$cucumberVersion")
testCompile("org.junit.jupiter:junit-jupiter-api:$junitJupiterVersion")
testRuntime("org.junit.jupiter:junit-jupiter-engine:$junitJupiterVersion")
```

Next we are going to create a `GET` request example using the GitHub API V3.

## GET

Example: How to list public email addresses for a user.

**Endpoint**

```bash
GET /user/public_emails
```

**Response**

```json
[
  {
    "email": "joseluis.delacruz@gmail.com",
    "verified": true,
    "primary": true,
    "visibility": "public"
  }
]
```

First, we are going to create our model definition:

```java
package com.jos.dem.webclient.model;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class PublicEmail {
  private String email;
  private boolean verified;
  private boolean primary;
  private String visibility;
}
```

Lombok is a great tool to avoid boilerplate code, for knowing more please go [here](https://projectlombok.org/). Next step is to create service definition:

```java
package com.jos.dem.webclient.service;

import reactor.core.publisher.Flux;
import com.jos.dem.webclient.model.SSHKey;
import com.jos.dem.webclient.model.PublicEmail;

public interface UserService {

  Flux<PublicEmail> getEmails();

}
```

Implementation:

```java
package com.jos.dem.webclient.service.impl;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.reactive.function.client.WebClient;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Flux;

import com.jos.dem.webclient.model.SSHKey;
import com.jos.dem.webclient.model.PublicEmail;
import com.jos.dem.webclient.service.UserService;

@Service
public class UserServiceImpl implements UserService {

  @Autowired
  private WebClient webClient;

  public Flux<PublicEmail> getEmails() {
    return webClient.get().uri("user/public_emails").retrieve()
    .bodyToFlux(PublicEmail.class);
  }

}
```

`WebClient` is a reactive client that provides an alternative to the `RestTemplate`. It exposes a functional, fluent API and relies on non-blocking I/O which allows it to support high concurrency more efficiently. For knowing more, please go to my previous WebClient post: [Spring Boot WebClient](/techtalk/spring/spring_boot_web_client)

```java
package com.jos.dem.webclient;

import static java.nio.charset.StandardCharsets.UTF_8;

import org.springframework.util.Base64Utils;
import org.springframework.boot.SpringApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.reactive.function.client.WebClient;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.PropertySource;

@SpringBootApplication
@PropertySource("classpath:application.properties")
public class WebClientApplication {

  @Value("${github.api.url}")
  private String githubApiUrl;
  @Value("${username}")
  private String username;
  @Value("${token}")
  private String token;

  @Bean
  public WebClient webClient() {
    return WebClient
      .builder()
        .baseUrl(githubApiUrl)
        .defaultHeader("Authorization", "Basic " + Base64Utils
          .encodeToString((username + ":" + token).getBytes(UTF_8)))
      .build();
  }

	public static void main(String[] args) {
		SpringApplication.run(WebClientApplication.class, args);
	}

}
```

This project is using Github’s [Basic Authentication](https://developer.github.com/v3/auth/#basic-authentication) and requires your Github username and access token that you can generate from here: [Personal Access Token](https://github.com/settings/tokens). Once you have that token you need to provide it to our Spring Boot project, this time we will use an `application.properties` file, please go to the [Project Configuration](https://github.com/josdem/webclient-workshop/wiki/Properties-File) for getting more information. The JUnit runner uses the JUnit framework to run the Cucumber Test. What we need is to create a single empty class with an annotation @RunWith(Cucumber.class) and define @CucumberOptions where we’re specifying the location of the Gherkin file which is also known as the feature file:

```java
package com.jos.dem.webclient;

import org.junit.runner.RunWith;
import cucumber.api.CucumberOptions;
import cucumber.api.junit.Cucumber;

@RunWith(Cucumber.class)
@CucumberOptions(features = "src/test/resources")
public class CucumberTest {}
```

Gherkin is a DSL language used to describe an application feature that needs to be tested. Here is our person Gherkin feature definition file src/test/resources/person.feature:

```gherkin
Feature: As a user I can get my public emails
  Scenario: User call to get his public emails
    Then User gets his public emails
```

The next step is to create a class with a user service so we can call our get email endpoint:

```java
package com.jos.dem.webclient;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.web.WebAppConfiguration;

import com.jos.dem.webclient.model.PublicEmail;
import com.jos.dem.webclient.service.UserService;

import reactor.core.publisher.Flux;

@ContextConfiguration(classes = WebClientApplication.class)
@WebAppConfiguration
public class UserIntegrationTest {

  @Autowired
  private UserService userService;

  Flux<PublicEmail> getEmails() throws Exception {
    return userService.getEmails();
  }

}
```

Now let’s create the method in the Java class to correspond to this test case scenario:

```java
package com.jos.dem.webclient;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assertions.assertEquals;

import com.jos.dem.webclient.model.PublicEmail;
import java.util.List;

import cucumber.api.java.en.Then;
import cucumber.api.java.en.When;
import reactor.core.publisher.Flux;

public class UserGetTest extends UserIntegrationTest {

  @Then("^User gets his public emails$")
  public void shouldGetEmails() throws Exception {
    List<PublicEmail> emails = getEmails()
      .collectList()
      .block();
    PublicEmail email = emails.get(0);

    assertTrue(emails.size() == 1,  () -> "Should be 1 email");
    assertAll("email",
        () -> assertEquals("joseluis.delacruz@gmail.com", email.getEmail(), "Should contains josdem's email"),
        () -> assertTrue(email.getVerified(), "Should be verified"),
        () -> assertTrue(email.getPrimary(), "Should be primary"),
        () -> assertEquals("public", email.getVisibility(), "Should be public")
    );
  }

}
```

## POST

Example: Create a label

**Endpoint**

```bash
POST /repos/:owner/:repo/labels
```

**Request**

```json
{
  "name": "cucumber",
  "description": "Cucumber is a very powerful testing framework written in the Ruby programming language",
  "color": "ed14c5"
}
```

**Response**

```json
{
  "id": 208045946,
  "node_id": "MDU6TGFiZWwyMDgwNDU5NDY=",
  "url": "https://api.github.com/repos/josdem/webclient-workshop/labels/cucumber",
  "name": "cucumber",
  "description": "Cucumber is a very powerful testing framework written in the Ruby programming language",
  "color": "ed14c5"
  "default": true
}
```

Label model definition

```java
package com.jos.dem.webclient.model;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Label {
  private String name;
  private String description;
  private String color;
}
```

Label service definition

```java
package com.jos.dem.webclient.service;

import reactor.core.publisher.Mono;
import com.jos.dem.webclient.model.LabelResponse;
import org.springframework.web.reactive.function.client.ClientResponse;

public interface LabelService {

  Mono<LabelResponse> create();

}
```

Label service implementation

```java
package com.jos.dem.webclient.service.impl;

import static org.springframework.http.MediaType.APPLICATION_JSON;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.reactive.function.client.WebClient;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Mono;

import javax.annotation.PostConstruct;

import com.jos.dem.webclient.model.Label;
import com.jos.dem.webclient.model.LabelResponse;
import com.jos.dem.webclient.service.LabelService;
import com.jos.dem.webclient.util.LabelCreator;

@Service
public class LabelServiceImpl implements LabelService {

  @Autowired
  private WebClient webClient;
  @Autowired
  private LabelCreator labelCreator;

  @Value("${github.labels.path}")
  private String githubLabelsPath;

  public Mono<LabelResponse> create() {
    return webClient.post()
      .uri(githubLabelsPath).accept(APPLICATION_JSON)
      .body(Mono.just(labelCreator.create()), Label.class)
      .retrieve()
      .bodyToMono(LabelResponse.class);
  }

}

```

Label creator is just a collaboratior helping to create label model and fill data on it, now let see create lable scenario definition.

```gherkin
Feature: As a user I want to create a label
  Scenario: User call to create new label with cucumer as a name
    Then User creates a new label
```

In the test side, we are going to create a label integration definition:

```java
package com.jos.dem.webclient;

import static org.junit.jupiter.api.Assertions.assertEquals;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.web.WebAppConfiguration;

import org.springframework.web.reactive.function.client.ClientResponse;

import com.jos.dem.webclient.model.LabelResponse;
import com.jos.dem.webclient.service.LabelService;

import reactor.core.publisher.Mono;

@ContextConfiguration(classes = WebClientApplication.class)
@WebAppConfiguration
public class LabelIntegrationTest {

  @Autowired
  private LabelService labelService;

  Mono<LabelResponse> create() throws Exception {
    return labelService.create();
  }

}
```

This is the Junit 5 test implementation

```java
package com.jos.dem.webclient;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertEquals;

import com.jos.dem.webclient.model.LabelResponse;
import cucumber.api.java.en.Then;

public class LabelPostTest extends LabelIntegrationTest {

  private LabelResponse response;

  @Then("^User creates a new label$")
  public void shouldCreateLabel() throws Exception {
    LabelResponse response = create()
      .block();

    assertAll("response",
      () -> assertEquals("cucumber", response.getName()),
      () -> assertEquals("ed14c5", response.getColor())
    );
  }

}
```

## PATCH

Example: Update a label

**Endpoint**

```bash
PATCH /repos/:owner/:repo/labels/:current_name
```

**Request**

```json
{
  "name": "spock",
  "description": "Spock is a testing and specification framework for Java and Groovy applications. It is beautiful and highly expressive",
  "color": "ff0000"
}
```

**Response**

```bash
Status: 200 OK
```

Label service definition updated:

```java
package com.jos.dem.webclient.service;

import reactor.core.publisher.Mono;
import com.jos.dem.webclient.model.LabelResponse;
import org.springframework.web.reactive.function.client.ClientResponse;

public interface LabelService {

  Mono<LabelResponse> create();
  Mono<ClientResponse> update(String name);

}
```

Label service implementation updated:

```java
package com.jos.dem.webclient.service.impl;

import static org.springframework.http.MediaType.APPLICATION_JSON;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.reactive.function.client.WebClient;
import org.springframework.web.reactive.function.client.ClientResponse;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Mono;

import javax.annotation.PostConstruct;

import com.jos.dem.webclient.model.Label;
import com.jos.dem.webclient.model.LabelResponse;
import com.jos.dem.webclient.service.LabelService;
import com.jos.dem.webclient.util.LabelCreator;

@Service
public class LabelServiceImpl implements LabelService {

  @Autowired
  private WebClient webClient;
  @Autowired
  private LabelCreator labelCreator;

  @Value("${github.labels.path}")
  private String githubLabelsPath;

  public Mono<LabelResponse> create() {
    return webClient.post()
      .uri(githubLabelsPath).accept(APPLICATION_JSON)
      .body(Mono.just(labelCreator.create()), Label.class)
      .retrieve()
      .bodyToMono(LabelResponse.class);
  }

  public Mono<ClientResponse> update(String name){
    return webClient.patch()
      .uri(githubLabelsPath + "/" + name).accept(APPLICATION_JSON)
      .body(Mono.just(labelCreator.update()), Label.class)
      .exchange();
  }

}
```

Feature definition updated:

```gherkin
Feature: As a user I want to create a label
  Scenario: User call to create new label with cucumer as a name
    Then User creates a new label
  Scenario: User call to update label to spock as a name
    Then User updates label
```

Label test integration client definition updated:

```java
package com.jos.dem.webclient;

import static org.junit.jupiter.api.Assertions.assertEquals;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.web.reactive.function.client.ClientResponse;

import com.jos.dem.webclient.model.LabelResponse;
import com.jos.dem.webclient.service.LabelService;

import reactor.core.publisher.Mono;

@ContextConfiguration(classes = WebClientApplication.class)
@WebAppConfiguration
public class LabelIntegrationTest {

  @Autowired
  private LabelService labelService;

  Mono<LabelResponse> create() throws Exception {
    return labelService.create();
  }

  Mono<ClientResponse> update(String name) throws Exception {
    return labelService.update(name);
  }

}
```

This is the Junit 5 test implementation updated:

```java
package com.jos.dem.webclient;

import static org.springframework.http.HttpStatus.OK;
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.springframework.web.reactive.function.client.ClientResponse;
import java.util.List;

import cucumber.api.java.en.Then;

public class LabelUpdateTest extends LabelIntegrationTest {

  @Then("^User updates label$")
  public void shouldCreateLabel() throws Exception {
    ClientResponse response = update("cucumber")
      .block();

    assertEquals(OK, response.statusCode(), "Should update to spock information");
  }

}
```

## DELETE

Example: Delete a label

**Endpoint**

```bash
DELETE /repos/:owner/:repo/labels/:name
```

**Response**

```bash
Status: 204 No Content
```

Label service definition updated:

```java
package com.jos.dem.webclient.service;

import reactor.core.publisher.Mono;
import com.jos.dem.webclient.model.LabelResponse;
import org.springframework.web.reactive.function.client.ClientResponse;

public interface LabelService {

  Mono<LabelResponse> create();
  Mono<ClientResponse> update(String name);
  Mono<ClientResponse> delete(String name);

}
```

Label service implementation updated:

```java
package com.jos.dem.webclient.service.impl;

import static org.springframework.http.MediaType.APPLICATION_JSON;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.reactive.function.client.WebClient;
import org.springframework.web.reactive.function.client.ClientResponse;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Mono;

import javax.annotation.PostConstruct;

import com.jos.dem.webclient.model.Label;
import com.jos.dem.webclient.model.LabelResponse;
import com.jos.dem.webclient.service.LabelService;
import com.jos.dem.webclient.util.LabelCreator;

@Service
public class LabelServiceImpl implements LabelService {

  @Autowired
  private WebClient webClient;
  @Autowired
  private LabelCreator labelCreator;

  @Value("${github.labels.path}")
  private String githubLabelsPath;

  public Mono<LabelResponse> create() {
    return webClient.post()
      .uri(githubLabelsPath).accept(APPLICATION_JSON)
      .body(Mono.just(labelCreator.create()), Label.class)
      .retrieve()
      .bodyToMono(LabelResponse.class);
  }

  public Mono<ClientResponse> update(String name){
    return webClient.patch()
      .uri(githubLabelsPath + "/" + name).accept(APPLICATION_JSON)
      .body(Mono.just(labelCreator.update()), Label.class)
      .exchange();
  }

  public Mono<ClientResponse> delete(String name){
    return webClient.delete()
      .uri(githubLabelsPath + "/" + name).accept(APPLICATION_JSON)
      .exchange();
  }

}
```

Feature definition updated:

```gherkin
Feature: As a user I want to create a label
  Scenario: User call to create new label with cucumer as a name
    Then User creates a new label
  Scenario: User call to update label to spock as a name
    Then User updates label
  Scenario: User call to delete a label spock as a name
    Then User deletes label
```

Label test integration client definition updated:

```java
package com.jos.dem.webclient;

import static org.junit.jupiter.api.Assertions.assertEquals;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.web.reactive.function.client.ClientResponse;

import com.jos.dem.webclient.model.LabelResponse;
import com.jos.dem.webclient.service.LabelService;

import reactor.core.publisher.Mono;

@ContextConfiguration(classes = WebClientApplication.class)
@WebAppConfiguration
public class LabelIntegrationTest {

  @Autowired
  private LabelService labelService;

  Mono<LabelResponse> create() throws Exception {
    return labelService.create();
  }

  Mono<ClientResponse> update(String name) throws Exception {
    return labelService.update(name);
  }

  Mono<ClientResponse> delete(String name) throws Exception {
    return labelService.delete(name);
  }

}
```

This is the Junit 5 test implementation updated:

```java
package com.jos.dem.webclient.service.impl;

import static org.springframework.http.MediaType.APPLICATION_JSON;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.reactive.function.client.WebClient;
import org.springframework.web.reactive.function.client.ClientResponse;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Mono;

import javax.annotation.PostConstruct;

import com.jos.dem.webclient.model.Label;
import com.jos.dem.webclient.model.LabelResponse;
import com.jos.dem.webclient.service.LabelService;
import com.jos.dem.webclient.util.LabelCreator;

@Service
public class LabelServiceImpl implements LabelService {

  @Autowired
  private WebClient webClient;
  @Autowired
  private LabelCreator labelCreator;

  @Value("${github.labels.path}")
  private String githubLabelsPath;

  public Mono<LabelResponse> create() {
    return webClient.post()
      .uri(githubLabelsPath).accept(APPLICATION_JSON)
      .body(Mono.just(labelCreator.create()), Label.class)
      .retrieve()
      .bodyToMono(LabelResponse.class);
  }

  public Mono<ClientResponse> update(String name){
    return webClient.patch()
      .uri(githubLabelsPath + "/" + name).accept(APPLICATION_JSON)
      .body(Mono.just(labelCreator.update()), Label.class)
      .exchange();
  }

  public Mono<ClientResponse> delete(String name){
    return webClient.delete()
      .uri(githubLabelsPath + "/" + name).accept(APPLICATION_JSON)
      .exchange();
  }

}
```

**Important:** Regarding to ordering feature test execution Cucumber features run in alphabetical order by feature file name.

**Using Maven**

You can do the same using Maven, the only difference is that you need to specify --build=maven parameter in the spring init command line:

```bash
spring init --dependencies=webflux,lombok --build=maven --language=java spring-boot-web-client
```

This is the `pom.xml` file generated along with Cucumber and Junit as dependencies on it added manualy:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem</groupId>
  <artifactId>webclient</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>webclient-workshop</name>
  <description>Shows how to integrate Cucumber to your Spring Boot application</description>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.3.RELEASE</version>
    <relativePath/>
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
    <cucumber.version>1.2.5</cucumber.version>
    <junit.jupiter.version>5.2.0</junit.jupiter.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-tomcat</artifactId>
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
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-api</artifactId>
      <version>${junit.jupiter.version}</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-engine</artifactId>
      <version>${junit.jupiter.version}</version>
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

To browse the project go [here](https://github.com/josdem/webclient-workshop), to download the project:

```bash
git clone https://github.com/josdem/webclient-workshop.git
```

To run the project with Gradle:

```bash
gradle test
```

To run the project with Maven:

```bash
mvn test
```


[Return to the main article](/techtalk/spring#Spring_Boot)
