+++
title =  "Webclient Cucumber and Junit 5"
description = "Webflux WebClient along with Cucumber and Junit 5 to create client test automation to the GitHub API v3"
date = "2018-07-09T20:31:06-04:00"
tags = ["josdem", "techtalks","programming","technology","spring boot", "webclient", "cucumber", "junit5"]
categories = ["techtalk", "code", "spring boot", "cucumber", "junit5"]
+++

This time I will show you how to combine Webflux [WebClient](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-webclient.html) along with [Cucumber](https://cucumber.io/) and [Junit 5](https://junit.org/junit5/) to create client test automation to the [GitHub API v3](https://developer.github.com/v3/?) Webflux [WebClient](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-webclient.html) along with [Cucumber](https://cucumber.io/) and [Junit 5](https://junit.org/junit5/) to create client test automation to the [GitHub API v3](https://developer.github.com/v3/?)

## GET

Example: we are going to list public email addresses for a user.

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

This project is using Github’s [Basic Authentication](https://developer.github.com/v3/auth/#basic-authentication) and requires your Github username and access token that you can generate from here: [Personal Access Token](https://github.com/settings/tokens). The JUnit runner uses the JUnit framework to run the Cucumber Test. What we need is to create a single empty class with an annotation @RunWith(Cucumber.class) and define @CucumberOptions where we’re specifying the location of the Gherkin file which is also known as the feature file:

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


To download the project:

```bash
git clone https://github.com/josdem/webclient-workshop.git
```

To run the project:

```bash
gradle test
```

[Return to the main article](/techtalk/spring#Spring_Boot)
