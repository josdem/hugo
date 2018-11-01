+++
title =  "Spring Webflux Server"
categories = ["techtalk", "code","spring webflux"]
tags = ["josdem", "techtalks","programming","technology", "Reactive Programming", "WebFlux", "Reactor", "Java","spring webflux"]
date = "2018-03-26T16:10:59-06:00"
description = "This post walks you through the process of creating endpoints using Spring Webflux routers."
+++

This post walks you through the process of creating endpoints using Spring Webflux. Please read this previous [Spring Webflux Basics](/techtalk/spring/spring_webflux_basics) before conitnue with this information. We are going to use `@RestController` to serve data from our `PersonRepository`. Let's consider the following example:

```java
package com.jos.dem.webflux.controller;

import reactor.core.publisher.Mono;
import reactor.core.publisher.Flux;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.webflux.model.Person;
import com.jos.dem.webflux.repository.PersonRepository;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@RestController
public class PersonController {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Autowired
  private PersonRepository personRepository;

  @GetMapping("/persons")
  public Flux<Person> findAll(){
    log.info("Calling find persons");
    return personRepository.findAll();
  }

  @GetMapping("/persons/{id}")
  public Mono<Person> findById(@PathVariable String id){
    log.info("Calling find person by id: " + id);
    return personRepository.findById(id);
  }

}
```

The original Spring Web MVC was running on Tomcat and was purpose built for the Servlet API and Servlet containers. The reactive web framework Spring WebFlux is fully non-blocking, supports Reactive Streams and runs on Netty server. `@RestController` indicates that we dont want to render views, but write the results straight into the response body instead.`@GetMapping` tells Spring that this is the place to route persons calls, it is the shorthand annotation for `@RequestMapping(method=RequestMethod.GET)`. So, after adding this controller class to your project and run Spring Boot, you can go to: [http://localhost:8080/persons](http://localhost:8080/persons)

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

Let me conclude by giving a short summary:

* Spring Boot now embraces reactive programming
* Using Spring MVC-style Controllers is a good start to adopting Reactive Web
* WebFlux combines well with Java functional programming
* WebFlux runs on Netty server by default

To run the project with Gradle:

```bash
gradle bootRun
```

To run the project with Maven:

```bash
mvn spring-boot:run
```

To browse the project go [here](https://github.com/josdem/reactive-webflux-workshop), to download the project:

```bash
git clone https://github.com/josdem/reactive-webflux-workshop.git
git fetch
git checkout feature/server
```


[Return to the main article](/techtalk/spring#Spring_Boot)