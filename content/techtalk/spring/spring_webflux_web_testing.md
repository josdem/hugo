+++
title =  "Spring Webflux Testing the Web Layer"
description = "Spring_webflux_web_testing"
date = "2019-02-03T13:00:31-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

In this technical post we will go through the process of testing a reactive web layer using [WebTestClient](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/web/reactive/server/WebTestClient.html). WebTestClient helps to test Spring [WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) controllers with auto configuration, if you want to know more about how to create Spring Webflux please go to my previous post getting started with Spring Webflux [here](/techtalk/spring/spring_webflux_basics). As an example target project to test let's use this one [Jugoterapia WebFlux](https://github.com/josdem/jugoterapia-webflux) which provides healthy juice and smoothie recipes. In this technical post we will review how to test the controllers in this project. Please consider this first controller.

```java
package com.jos.dem.jugoterapia.webflux.controller;

import reactor.core.publisher.Mono;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Api(tags={"knows how to respond to sanity checks"})
@RestController
@RequestMapping("/health")
public class HealthController {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @ApiImplicitParam(name = "ping", value = "Ping message", required = true, dataType = "string", paramType = "path")
  @GetMapping("/{ping}")
  public Mono<String> check(@PathVariable("ping") String ping){
    log.info(ping);
    return Mono.just("pong");
  }

}
```

The responsability in this conrtoller is to provide a health check, this is the test case we have for it.

```java
package com.jos.dem.jugoterapia.webflux;

import static org.springframework.http.MediaType.APPLICATION_JSON;

import org.junit.Test;
import org.junit.runner.RunWith;

import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.web.reactive.server.WebTestClient;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class HealthControllerTest {

  @Autowired
  private WebTestClient webClient;

  @Test
  public void shouldGetPong() throws Exception {
    webClient.get().uri("/health/{ping}", "ping").accept(APPLICATION_JSON)
      .exchange()
		  .expectStatus().isOk()
      .expectBody(String.class).isEqualTo("pong");
  }

}
```

Since Jugoterapia Webflux is a Spring Boot application we are using `@SpringBootTest` annotation that can be specified on a test class that runs Spring Boot based tests, also we are using `WebEnvironment` which creates a reactive web application context listening on a random port.

```java
package com.jos.dem.jugoterapia.webflux.controller;

import reactor.core.publisher.Flux;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.jugoterapia.webflux.model.Category;
import com.jos.dem.jugoterapia.webflux.model.Beverage;
import com.jos.dem.jugoterapia.webflux.util.LanguageResolver;
import com.jos.dem.jugoterapia.webflux.service.CategoryService;
import com.jos.dem.jugoterapia.webflux.service.BeverageService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@RestController
@RequestMapping("/categories")
public class CategoryController {

  @Autowired
  private CategoryService categoryService;
  @Autowired
  private BeverageService beverageService;
  @Autowired
  private LanguageResolver languageResolver;

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @GetMapping("/{language}")
  public Flux<Category> getCategories(@PathVariable("language") String language){
    log.info("Listing categories");
    return categoryService.findByI18n(languageResolver.resolve(language));
  }

  @GetMapping(value="/{id}/beverages")
  public Flux<Beverage> getBeverages(@PathVariable("id") Integer categoryId){
    log.info("Listing beverages by category: {}", categoryId);
    return beverageService.findByCategoryId(categoryId);
  }

}
```

This controller get juice categories by languge and beverages by category id, and here is the test case

```java
package com.jos.dem.jugoterapia.webflux;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.http.MediaType.APPLICATION_JSON_UTF8;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.junit4.SpringRunner;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.web.reactive.server.WebTestClient;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;

import com.jos.dem.jugoterapia.webflux.model.Category;
import com.jos.dem.jugoterapia.webflux.model.Beverage;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.DEFINED_PORT)
public class CategoryControllerTest {

  @Autowired
  private WebTestClient webClient;

  @Test
  public void shouldGetCategoriesByLanguage() throws Exception {
    webClient.get().uri("/categories/{language}", "es").accept(APPLICATION_JSON)
      .exchange()
      .expectStatus().isOk()
      .expectHeader().contentType(APPLICATION_JSON_UTF8)
      .expectBodyList(Category.class);
  }

  @Test
  public void shouldGetBeveragesByCategory() throws Exception {
    webClient.get().uri("/categories/{id}/beverages", 1).accept(APPLICATION_JSON)
      .exchange()
      .expectStatus().isOk()
      .expectHeader().contentType(APPLICATION_JSON_UTF8)
      .expectBodyList(Beverage.class);
  }

}
```

Finally we have a beverage controller which gets a beverage by id.

```java
package com.jos.dem.jugoterapia.webflux.controller;

import reactor.core.publisher.Mono;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.jugoterapia.webflux.model.Beverage;
import com.jos.dem.jugoterapia.webflux.service.BeverageService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@RequestMapping("/beverages")
public class BeverageController {

  @Autowired
  private BeverageService beverageService;

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @GetMapping("/{id}")
  public Mono<Beverage> getBeverage(@PathVariable("id") Integer beverageId){
    log.info("Listing beverages by id: {}", beverageId);
    return beverageService.findById(beverageId);
  }

}
```

Here is the test case

```java
package com.jos.dem.jugoterapia.webflux;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.http.MediaType.APPLICATION_JSON;
import static org.springframework.http.MediaType.APPLICATION_JSON_UTF8;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.junit4.SpringRunner;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.web.reactive.server.WebTestClient;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;

import com.jos.dem.jugoterapia.webflux.model.Beverage;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.DEFINED_PORT)
public class BeverageControllerTest {

  @Autowired
  private WebTestClient webClient;

  @Test
  public void shouldGetBeverage() throws Exception {
    webClient.get().uri("/beverages/{id}", 83).accept(APPLICATION_JSON)
      .exchange()
      .expectStatus().isOk()
      .expectHeader().contentType(APPLICATION_JSON_UTF8)
      .expectBody(Beverage.class);
  }

}
```

To browse the complete project go [here](https://github.com/josdem/jugoterapia-webflux), to download the project:

```bash
git clone git@github.com:josdem/jugoterapia-webflux.git
```

To run the project:

```bash
gradle bootRun
```

To test the project:

```bash
gradle test
```

[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
