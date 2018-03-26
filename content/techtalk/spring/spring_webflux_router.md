+++
title =  "Spring Webflux Router"
date = "2018-03-26T16:10:59-06:00"
tags = ["josdem", "techtalks","programming","technology", "Reactive Programming", "WebFlux", "Reactor", "Java"]
categories = ["techtalk", "code"]
+++

This post walks you through the process of creating endpoints using Spring Webflux routers. Please read this previous [post](/techtalk/spring/spring_webflux_basics) before conitnue with this information.

In Spring MVC we are using `@Controller` in WebFlux we are using:

**Router Function**

Incoming requests are routed to handler functions with a `RouterFunction<T>` it has a similar purpose as a `@RequestMapping` annotation.

Let's consider the following example:

```java
package com.jos.dem.webflux.config;

import static org.springframework.web.reactive.function.server.RequestPredicates.GET;

import com.jos.dem.webflux.model.Person;
import com.jos.dem.webflux.repository.PersonRepository;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.function.server.RouterFunction;
import org.springframework.web.reactive.function.server.ServerResponse;
import org.springframework.web.reactive.function.server.RouterFunctions;

@Configuration
public class WebConfig {

  @Autowired
  private PersonRepository personRepository;

  @Bean
  public RouterFunction<ServerResponse> routes(){
    return RouterFunctions
      .route(GET("/persons"),
      request -> ServerResponse.ok().body(personRepository.findAll(), Person.class))
      .andRoute(GET("/persons/{id}"), 
      request -> ServerResponse.ok().body(personRepository.findById(request.pathVariable("id")), Person.class));
  }

}
```

The original Spring Web MVC was running on Tomcat and was purpose built for the Servlet API and Servlet containers. The reactive web framework Spring WebFlux is fully non-blocking, supports Reactive Streams and runs on Netty server, but at the end from the client perspective we hit them in the same way.

So, after adding this configuration class to your project and run Spring Boot, you can go to: [http://localhost:8080/persons](http://localhost:8080/persons)

```json
[
  {
    "uuid": "7baac840-0ff1-4eea-a1f3-78a6320f5bcc",
    "nickname": "edzero",
    "email": "edzero@email.com"
  },
  {
    "uuid": "7b098510-e260-4dc2-8b15-24fc403eb939",
    "nickname": "skuarch",
    "email": "skuarch@email.com"
  },
  {
    "uuid": "d6089e68-e8c4-4ebe-aa2f-b566e13856a8",
    "nickname": "siedrix",
    "email": "siedrix@email.com"
  },
  {
    "uuid": "ac55dfb8-795d-48d9-9827-4e401c0e4853",
    "nickname": "tgrip",
    "email": "tgrip@email.com"
  },
  {
    "uuid": "5904ff32-7e36-4a81-b90f-c0d7a91214eb",
    "nickname": "josdem",
    "email": "josdem@email.com"
  }
]
```

And this one is the endpoint to get a person by id: [http://localhost:8080/persons/5904ff32-7e36-4a81-b90f-c0d7a91214eb](http://localhost:8080/persons/5904ff32-7e36-4a81-b90f-c0d7a91214eb)

```json
{
  "uuid": "5904ff32-7e36-4a81-b90f-c0d7a91214eb",
  "nickname": "josdem",
  "email": "josdem@email.com"
}
```

**Conclusion**

This ends the introduction to Spring’s new functional style web framework. Let me conclude by giving a short summary:

* Spring Boot now embraces reactive programming
* Router functions route to handler functions
* Router functions can be run in a reactive web runtime
* WebFlux combines well with Java functional programming
* WebFlux runs on Netty server by default

To download the project:

```bash
git clone https://github.com/josdem/reactive-webflux-workshop.git
git fetch
git checkout feature/router
```

To run the project:

```bash
gradle bootRun
```


[Return to the main article](/techtalk/spring)
