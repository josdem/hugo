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

This project is using Github’s [Basic Authentication](https://developer.github.com/v3/auth/#basic-authentication) and requires your Github username and access token that you can generate from here: [Personal Access Token](https://github.com/settings/tokens).



To download the project:

```bash
git clone https://github.com/josdem/webclient-workshop.git
```

To run the project:

```bash
gradle test
```

[Return to the main article](/techtalk/spring#Spring_Boot)
