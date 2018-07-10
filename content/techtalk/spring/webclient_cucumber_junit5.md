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


To download the project:

```bash
git clone https://github.com/josdem/webclient-workshop.git
```

To run the project:

```bash
gradle test
```

[Return to the main article](/techtalk/spring#Spring_Boot)
