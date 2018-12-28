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

[Return to the main article](/techtalk/spring#Spring_Boot)
