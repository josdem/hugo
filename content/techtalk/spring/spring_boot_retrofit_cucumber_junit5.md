+++
title =  "Spring Boot Retrofit2 Cucumber & Junit5"
description = "Spring_boot_retrofit_cucumber_junit5"
date = "2018-12-28T16:13:39-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

This time I will show you how to combine [Retrofit](https://square.github.io/retrofit/) along with [Cucumber](https://cucumber.io/) and [Junit 5](https://junit.org/junit5/) in order to consume [GitHub API v3](https://developer.github.com/v3/?) public REST API. First, let’s start creating a new Spring Boot project with Webflux as dependency:

```bash
spring init --dependencies=webflux --build=gradle --language=java retrofit-workshop
```

**NOTE:** If you need to know what tools you need to have installed in your computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot)


Here is the complete `build.gradle` file generated:

```groovy
buildscript {
  ext {
    springBootVersion = '2.1.1.RELEASE'
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

group = 'com.jos.dem.retrofit.workshop'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
  mavenCentral()
}

dependencies {
  compile('org.springframework.boot:spring-boot-starter-webflux')
  compile('org.springframework.boot:spring-boot-starter-tomcat')
  testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

Now add latest Retrofit, Cucumber and Junit 5 Framework dependencies to your `build.gradle` file:

```groovy
implementation("com.squareup.retrofit2:retrofit:$retrofitVersion")
compile('com.squareup.retrofit2:converter-gson:2.5.0')
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
package com.jos.dem.retrofit.workshop.model;

public class PublicEmail {
  private String email;
  private boolean verified;
  private boolean primary;
  private String visibility;

  public String getEmail(){
    return email;
  }

  public boolean isVerified(){
    return verified;
  }

  public boolean isPrimary(){
    return primary;
  }

  public String getVisibility(){
    return visibility;
  }

}
```

Next step is to create service definition:

```java
package com.jos.dem.retrofit.workshop.service;

import java.util.List;

import retrofit2.Call;
import retrofit2.http.GET;

import com.jos.dem.retrofit.workshop.model.PublicEmail;

public interface UserService {

  @GET("user/public_emails")
  Call<List<PublicEmail>> getEmails();

}
```

Implementation:

```java
package com.jos.dem.retrofit.workshop.service.impl;

import java.util.List;

import retrofit2.Call;
import retrofit2.Retrofit;

import javax.annotation.PostConstruct;

import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.retrofit.workshop.model.PublicEmail;
import com.jos.dem.retrofit.workshop.service.UserService;

@Service
public class UserServiceImpl implements UserService {

  @Autowired
  private Retrofit retrofit;

  private UserService userService;

  @PostConstruct
  public void setup() {
    userService = retrofit.create(UserService.class);
  }

  public Call<List<PublicEmail>> getEmails() {
    return userService.getEmails();
  }

}
```

Retrofit turns your HTTP API into a Java interface and in the `@PostConstruct` Retrofit is creating UserService based in that interface.

```java
package com.jos.dem.retrofit.workshop;

import java.io.IOException;

import org.springframework.boot.SpringApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.PropertySource;

import retrofit2.Retrofit;
import retrofit2.converter.gson.GsonConverterFactory;

import okhttp3.Request;
import okhttp3.Response;
import okhttp3.Interceptor;
import okhttp3.OkHttpClient;

@SpringBootApplication
@PropertySource("classpath:application.properties")
public class RetrofitWorkshopApplication {

  @Value("${github.api.url}")
  private String githubApiUrl;
  @Value("${token}")
  private String token;

  OkHttpClient.Builder clientBuilder = new OkHttpClient.Builder();

  Interceptor interceptor = new Interceptor() {
    @Override
    public okhttp3.Response intercept(Chain chain) throws IOException {
      Request request = chain.request().newBuilder().addHeader("Authorization", "token " + token).build();
      return chain.proceed(request);
    }
  };

  @Bean
  public Retrofit retrofit() {
    clientBuilder.interceptors().add(interceptor);
    return new Retrofit.Builder()
      .baseUrl(githubApiUrl)
      .client(clientBuilder.build())
      .addConverterFactory(GsonConverterFactory.create())
      .build();
  }

  public static void main(String[] args) {
    SpringApplication.run(RetrofitWorkshopApplication.class, args);
  }

}
```

Here we are using the Authorization header in order to set the API Github token. This strategy is using a Interceptor with request headers. This project is using Github’s [Basic Authentication](https://developer.github.com/v3/auth/#basic-authentication) and requires your Github username and access token that you can generate from here: [Personal Access Token](https://github.com/settings/tokens). Once you have that token you need to provide it to our Spring Boot project, this time we will use an `application.properties` file, please go to the [Project Configuration](https://github.com/josdem/retrofit-workshop/wiki/Properties-File) for getting more information. The JUnit runner uses the JUnit framework to run the Cucumber Test. What we need is to create a single empty class with an annotation @RunWith(Cucumber.class) and define @CucumberOptions where we’re specifying the location of the Gherkin file which is also known as the feature file:

```java
package com.jos.dem.retrofit.workshop;

import org.junit.runner.RunWith;
import cucumber.api.CucumberOptions;
import cucumber.api.junit.Cucumber;

@RunWith(Cucumber.class)
@CucumberOptions(features = "src/test/resources")
public class CucumberTest {}
```

Gherkin is a DSL language used to describe an application feature that needs to be tested. Here is our person Gherkin feature definition file: `src/test/resources/person.feature`

```gherkin
Feature: As a user I can get my public information
  Scenario: User call to get his public emails
    Then User gets his public emails
```

The next step is to create an abstraction for our user service test so we can call our get email endpoint:

```java
package com.jos.dem.retrofit.workshop;

import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.web.WebAppConfiguration;

@ContextConfiguration(classes = RetrofitWorkshopApplication.class)
@WebAppConfiguration
public class UserIntegrationTest {}
```

Now let’s create the method in the Java class to correspond to this test case scenario:

```java
package com.jos.dem.retrofit.workshop;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.List;

import cucumber.api.java.After;
import cucumber.api.java.Before;
import cucumber.api.java.en.Then;
import cucumber.api.java.en.When;

import org.springframework.beans.factory.annotation.Autowired;

import retrofit2.Call;
import retrofit2.Response;

import com.jos.dem.retrofit.workshop.model.SSHKey;
import com.jos.dem.retrofit.workshop.model.PublicEmail;
import com.jos.dem.retrofit.workshop.service.UserService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class UserTest extends UserIntegrationTest {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Autowired
  private UserService userService;

  @Before
  public void setup() {
    log.info("Before any test execution");
  }

  @Then("^User gets his public emails$")
  public void shouldGetEmails() throws Exception {
    log.info("Validating collection integrity");

    Call<List<PublicEmail>> call = userService.getEmails();
    Response<List<PublicEmail>> response = call.execute();
    List<PublicEmail> emails = response.body();

    assertFalse(emails.isEmpty(), () -> "Should not be empty");
    assertTrue(emails.size() == 1,  () -> "Should be 1 email");

    PublicEmail email = emails.get(0);
    log.info("Validating email attributes");
      assertAll("email",
        () -> assertEquals("joseluis.delacruz@gmail.com", email.getEmail(), "Should contains josdem's email"),
        () -> assertTrue(email.isVerified(), "Should be verified"),
        () -> assertTrue(email.isPrimary(), "Should be primary"),
        () -> assertEquals("public", email.getVisibility(), "Should be public")
      );
  }

  @After
  public void tearDown() {
    log.info("After all test execution");
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
package com.jos.dem.retrofit.workshop.model;

public class Label {

  private String name;
  private String description;
  private String color;

  public Label(String name, String description, String color){
    this.name = name;
    this.description = description;
    this.color = color;
  }

  public void setName(String name){
    this.name = name;
  }

  public String getName(){
    return name;
  }

  public void setDescription(String description){
    this.description = description;
  }

  public String getDescription(){
    return description;
  }

  public void setColor(String color){
    this.color = color;
  }

  public String getColor(){
    return color;
  }

}
```

Label service definition

```java
package com.jos.dem.retrofit.workshop.service;

import retrofit2.Call;
import retrofit2.Response;
import retrofit2.http.Body;
import retrofit2.http.POST;

import com.jos.dem.retrofit.workshop.model.Label;
import com.jos.dem.retrofit.workshop.model.LabelResponse;

public interface LabelService {

  @POST("repos/josdem/retrofit-workshop/labels")
  Call<LabelResponse> create(@Body Label label);

}
```

Label service implementation

```java
package com.jos.dem.retrofit.workshop.service.impl;

import retrofit2.Call;
import retrofit2.Response;
import retrofit2.Retrofit;
import retrofit2.http.Body;
import retrofit2.http.Path;


import javax.annotation.PostConstruct;

import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.retrofit.workshop.model.Label;
import com.jos.dem.retrofit.workshop.model.LabelResponse;
import com.jos.dem.retrofit.workshop.service.LabelService;

@Service
public class LabelServiceImpl implements LabelService {

  @Autowired
  private Retrofit retrofit;

  private LabelService labelService;

  @PostConstruct
  public void setup() {
    labelService = retrofit.create(LabelService.class);
  }

  public Call<LabelResponse> create(@Body Label label) {
    return labelService.create(label);
  }

}
```

```gherkin
Feature: As a user I want to create a label
  Scenario: User call to create new label with cucumer as a name
    Then User creates a new label
```

This is the Junit 5 test implementation

```java
package com.jos.dem.retrofit.workshop;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertEquals;

import cucumber.api.java.After;
import cucumber.api.java.Before;
import cucumber.api.java.en.Then;

import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;

import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.retrofit.workshop.model.LabelResponse;
import com.jos.dem.retrofit.workshop.service.LabelService;
import com.jos.dem.retrofit.workshop.util.LabelCreator;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LabelCreateTest extends LabelIntegrationTest {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Autowired
  private LabelService labelService;
  @Autowired
  private LabelCreator labelCreator;

  @Before
  public void setup() {
    log.info("Before any test execution");
  }

  @Then("User creates a new label")
  public void shouldCreateLabel() throws Exception {
    log.info("Running: User creates a new label");

    Call<LabelResponse> call = labelService.create(labelCreator.create());
    Response<LabelResponse> response = call.execute();
    LabelResponse label = response.body();

    assertAll("response",
      () -> assertEquals("cucumber", label.getName()),
      () -> assertEquals("ed14c5", label.getColor())
    );
  }

  @After
  public void tearDown() {
    log.info("After all test execution");
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
package com.jos.dem.retrofit.workshop.service;

import retrofit2.Call;
import retrofit2.Response;
import retrofit2.http.Body;
import retrofit2.http.Path;
import retrofit2.http.POST;
import retrofit2.http.PATCH;

import com.jos.dem.retrofit.workshop.model.Label;
import com.jos.dem.retrofit.workshop.model.LabelResponse;

public interface LabelService {

  @POST("repos/josdem/retrofit-workshop/labels")
  Call<LabelResponse> create(@Body Label label);
  @PATCH("repos/josdem/retrofit-workshop/labels/{name}")
  Call<LabelResponse> update(@Body Label label, @Path("name") String name);

}

```

Label service implementation updated:


```java
package com.jos.dem.retrofit.workshop.service.impl;

import retrofit2.Call;
import retrofit2.Response;
import retrofit2.Retrofit;
import retrofit2.http.Body;
import retrofit2.http.Path;


import javax.annotation.PostConstruct;

import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.retrofit.workshop.model.Label;
import com.jos.dem.retrofit.workshop.model.LabelResponse;
import com.jos.dem.retrofit.workshop.service.LabelService;

@Service
public class LabelServiceImpl implements LabelService {

  @Autowired
  private Retrofit retrofit;

  private LabelService labelService;

  @PostConstruct
  public void setup() {
    labelService = retrofit.create(LabelService.class);
  }

  public Call<LabelResponse> create(@Body Label label) {
    return labelService.create(label);
  }

  public Call<LabelResponse> update(@Body Label label, @Path("name") String name) {
    return labelService.update(label, name);
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

Label update test definition:

```java
package com.jos.dem.retrofit.workshop;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertEquals;

import cucumber.api.java.After;
import cucumber.api.java.Before;
import cucumber.api.java.en.Then;

import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;

import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.retrofit.workshop.model.LabelResponse;
import com.jos.dem.retrofit.workshop.service.LabelService;
import com.jos.dem.retrofit.workshop.util.LabelCreator;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LabelUpdateTest extends LabelIntegrationTest {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Autowired
  private LabelService labelService;
  @Autowired
  private LabelCreator labelCreator;

  @Before
  public void setup() {
    log.info("Before any test execution");
  }

  @Then("User updates label")
  public void shouldUpdateLabel() throws Exception {
    log.info("Running: User updates label");

    Call<LabelResponse> call = labelService.update(labelCreator.update(), "cucumber");
    Response<LabelResponse> response = call.execute();
    LabelResponse label = response.body();

    assertAll("response",
      () -> assertEquals("spock", label.getName()),
      () -> assertEquals("ff0000", label.getColor())
    );
  }

  @After
  public void tearDown() {
    log.info("After all test execution");
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
package com.jos.dem.retrofit.workshop.service;

import retrofit2.Call;
import retrofit2.Response;
import retrofit2.http.Body;
import retrofit2.http.Path;
import retrofit2.http.POST;
import retrofit2.http.PATCH;
import retrofit2.http.DELETE;

import com.jos.dem.retrofit.workshop.model.Label;
import com.jos.dem.retrofit.workshop.model.LabelResponse;

public interface LabelService {

  @POST("repos/josdem/retrofit-workshop/labels")
  Call<LabelResponse> create(@Body Label label);
  @PATCH("repos/josdem/retrofit-workshop/labels/{name}")
  Call<LabelResponse> update(@Body Label label, @Path("name") String name);
  @DELETE("repos/josdem/retrofit-workshop/labels/{name}")
  Call<Response<Void>> delete(@Path("name") String name);

}
```

Label service implementation updated:

```java
package com.jos.dem.retrofit.workshop.service.impl;

import retrofit2.Call;
import retrofit2.Response;
import retrofit2.Retrofit;
import retrofit2.http.Body;
import retrofit2.http.Path;


import javax.annotation.PostConstruct;

import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.retrofit.workshop.model.Label;
import com.jos.dem.retrofit.workshop.model.LabelResponse;
import com.jos.dem.retrofit.workshop.service.LabelService;

@Service
public class LabelServiceImpl implements LabelService {

  @Autowired
  private Retrofit retrofit;

  private LabelService labelService;

  @PostConstruct
  public void setup() {
    labelService = retrofit.create(LabelService.class);
  }

  public Call<LabelResponse> create(@Body Label label) {
    return labelService.create(label);
  }

  public Call<LabelResponse> update(@Body Label label, @Path("name") String name) {
    return labelService.update(label, name);
  }

  public Call<Response<Void>> delete(@Path("name") String name) {
    return labelService.delete(name);
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

Label delete test definition:

```java
package com.jos.dem.retrofit.workshop;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertEquals;

import cucumber.api.java.After;
import cucumber.api.java.Before;
import cucumber.api.java.en.Then;

import retrofit2.Call;
import retrofit2.Callback;
import retrofit2.Response;

import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.retrofit.workshop.model.LabelResponse;
import com.jos.dem.retrofit.workshop.service.LabelService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LabelDeleteTest extends LabelIntegrationTest {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Autowired
  private LabelService labelService;

  @Before
  public void setup() {
    log.info("Before any test execution");
  }

  @Then("User deletes label")
  public void shouldUpdateLabel() throws Exception {
    log.info("Running: User updates label");

    Call<Response<Void>> call = labelService.delete("spock");
    Response<Response<Void>> response = call.execute();
    assertEquals(204, response.code());

  }

  @After
  public void tearDown() {
    log.info("After all test execution");
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
  <artifactId>retrofit</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>retorfit-workshop</name>
  <description>Retrofit configuration and examples to consume GitHub API v3</description>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.1.RELEASE</version>
    <relativePath/>
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <maven.surefire.version>2.18</maven.surefire.version>
    <maven-compiler.version>3.8.0</maven-compiler.version>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <java.version>1.8</java.version>
    <cucumber.version>1.2.5</cucumber.version>
    <junit.jupiter.version>5.3.1</junit.jupiter.version>
    <retrofit.version>2.5.0</retrofit.version>
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
      <groupId>com.squareup.retrofit2</groupId>
      <artifactId>retrofit</artifactId>
      <version>${retrofit.version}</version>
    </dependency>
    <dependency>
      <groupId>com.squareup.retrofit2</groupId>
      <artifactId>converter-gson</artifactId>
      <version>${retrofit.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
      <version>3.7</version>
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
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.0</version>
        <configuration>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>${maven.surefire.version}</version>
      </plugin>
    </plugins>
  </build>

</project>
```

To browse the project go [here](https://github.com/josdem/retrofit-workshop), to download the project:

```bash
git clone https://github.com/josdem/retrofit-workshop.git
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
