+++
title =  "Spring Webflux Testing the Web Layer"
description = "Spring webflux web testing"
date = "2019-02-03T13:00:31-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

In this technical post we will go through the process of testing a reactive web layer using [WebTestClient](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/web/reactive/server/WebTestClient.html). WebTestClient helps to test Spring [WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) controllers with auto configuration, if you want to know more about how to create Spring Webflux please go to my previous post getting started with Spring Webflux [here](/techtalk/spring/spring_webflux_basics). As project target to test we will use [Jugoterapia WebFlux](https://github.com/josdem/jugoterapia-webflux) which provides healthy juice and smoothie recipes. Please consider this first controller.

```java
package com.jos.dem.jugoterapia.webflux.controller;

import org.springframework.http.MediaType;
import reactor.core.publisher.Mono;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Api(tags={"knows how to respond to health checks"})
@RestController
@RequestMapping("/health")
public class HealthController {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @ApiImplicitParam(name = "ping", value = "Ping message", required = true, dataType = "string", paramType = "path")
  @GetMapping(value = "/{ping}", produces = MediaType.APPLICATION_JSON_VALUE)
  public Mono<String> check(@PathVariable("ping") String ping){
    log.info(ping);
    return Mono.just("pong");
  }

}
```

The responsability in this conrtoller is to validate that our API is up and running!, this is the test case.

```java
package com.jos.dem.jugoterapia.webflux;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.test.web.reactive.server.WebTestClient;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class HealthControllerTest {

  @Autowired
  private WebTestClient webClient;

  @Test
  @DisplayName("Should get pong")
  void shouldGetPong() throws Exception {
    webClient.get().uri("/health/{ping}", "ping")
            .exchange()
            .expectStatus().isOk()
            .expectBody(String.class).isEqualTo("pong");
  }

}
```

Using `@SpringBootTest` annotation we specify a Spring Boot based tests, also we are using `WebEnvironment` which creates a reactive web application context listening on a random port.

```java
package com.jos.dem.jugoterapia.webflux.controller;

import reactor.core.publisher.Flux;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.beans.factory.annotation.Autowired;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;

import com.jos.dem.jugoterapia.webflux.model.Category;
import com.jos.dem.jugoterapia.webflux.model.Beverage;
import com.jos.dem.jugoterapia.webflux.util.LanguageResolver;
import com.jos.dem.jugoterapia.webflux.service.CategoryService;
import com.jos.dem.jugoterapia.webflux.service.BeverageService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Api(tags={"knows how receive manage category requests"})
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

  @GetMapping("/")
  public Flux<Category> getCategories(){
    log.info("Listing categories");
    return categoryService.findByI18n("es");
  }

  @ApiImplicitParam(name = "language", value = "Language required", required = true, dataType = "string", paramType = "path")
  @GetMapping("/{language}")
  public Flux<Category> getCategories(@PathVariable("language") String language){
    log.info("Listing categories");
    return categoryService.findByI18n(languageResolver.resolve(language));
  }

  @ApiImplicitParam(name = "id", value = "Category's id", required = true, dataType = "int", paramType = "path")
  @GetMapping(value="/{id}/beverages")
  public Flux<Beverage> getBeverages(@PathVariable("id") Integer categoryId){
    log.info("Listing beverages by category: {}", categoryId);
    return beverageService.findByCategoryId(categoryId);
  }

}
```

This controller get juice categories by language and beverages by category id, and here is the test case

```java
package com.jos.dem.jugoterapia.webflux;

import com.jos.dem.jugoterapia.webflux.model.Beverage;
import com.jos.dem.jugoterapia.webflux.model.Category;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.test.web.reactive.server.WebTestClient;

import static org.hamcrest.CoreMatchers.equalTo;
import static org.springframework.http.MediaType.*;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class CategoryControllerTest {

  @Autowired
  private WebTestClient webClient;

  @Test
  @DisplayName("Should get all categories")
  void shouldGetCategories() throws Exception {
    webClient.get().uri("/categories/")
            .exchange()
            .expectStatus().isOk()
            .expectHeader().contentType(APPLICATION_JSON_VALUE)
            .expectBodyList(Category.class);
  }

  @Test
  @DisplayName("Should get categories in spanish")
  void shouldGetCategoriesByLanguage() throws Exception {
    webClient.get().uri("/categories/{language}", "es")
            .exchange()
            .expectStatus().isOk()
            .expectHeader().contentType(APPLICATION_JSON_VALUE)
            .expectBodyList(Category.class)
            .value(categories -> categories.size(), equalTo(4))
            .value(categories -> categories.get(0).getName(), equalTo("Curativos"))
            .value(categories -> categories.get(1).getName(), equalTo("Energizantes"))
            .value(categories -> categories.get(2).getName(), equalTo("Saludables"))
            .value(categories -> categories.get(3).getName(), equalTo("Estimulantes"));
  }

  @Test
  @DisplayName("Should get categories by id")
  public void shouldGetBeveragesByCategory() throws Exception {
    webClient.get().uri("/categories/{id}/beverages", 1)
            .exchange()
            .expectStatus().isOk()
            .expectHeader().contentType(APPLICATION_JSON_VALUE)
            .expectBodyList(Beverage.class);
  }

}
```

In the test case `shouldGetCategoriesByLanguage()` we are validating our list size and list content. Finally we have a beverage controller which gets a beverage by id and ingredients by keyword.

```java
package com.jos.dem.jugoterapia.webflux.controller;

import reactor.core.publisher.Mono;
import reactor.core.publisher.Flux;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.beans.factory.annotation.Autowired;

import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;

import com.jos.dem.jugoterapia.webflux.model.Beverage;
import com.jos.dem.jugoterapia.webflux.service.BeverageService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Api(tags = {"knows how receive manage beverage requests"})
@RestController
@RequestMapping("/beverages")
public class BeverageController {

  @Autowired
  private BeverageService beverageService;

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @ApiImplicitParam(name = "id", value = "Beverage's id", required = true, dataType = "int", paramType = "path")
  @GetMapping("/{id}")
  public Mono<Beverage> getBeverage(@PathVariable("id") Integer beverageId){
    log.info("Listing beverages by id: {}", beverageId);
    return beverageService.findById(beverageId);
  }

  @ApiImplicitParam(name = "keyword", value = "Beverage ingredients contain keyword", required = true, dataType = "string", paramType = "path")
  @GetMapping("/ingredients/{keyword}")
  public Flux<Beverage> getBeverageByKeyword(@PathVariable("keyword") String keyword){
    log.info("Listing beverages where ingredients contains: {}", keyword);
    return beverageService.findByIngredientKeyword(keyword);
  }

}
```

Here is the test case

```java
package com.jos.dem.jugoterapia.webflux;

import com.jos.dem.jugoterapia.webflux.model.Beverage;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.test.web.reactive.server.WebTestClient;

import static org.hamcrest.CoreMatchers.*;
import static org.springframework.http.MediaType.APPLICATION_JSON_VALUE;
import static org.junit.jupiter.api.Assertions.assertTrue;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class BeverageControllerTest {

  @Value("${bucket.url}")
  private String bucketUrl;

  @Autowired
  private WebTestClient webClient;

  @Test
  @DisplayName("Should get beverage")
  void shouldGetBeverage() {
    webClient.get().uri("/beverages/{id}", 83)
            .exchange()
            .expectStatus().isOk()
            .expectHeader().contentType(APPLICATION_JSON_VALUE)
            .expectBody(Beverage.class)
            .value(beverage -> beverage.getName(), equalTo("Nutritive Carrot Smoothie"))
            .value(beverage -> beverage.getIngredients(), equalTo("4 Carrots,1 Celery Stalk,1 Pear,10 Spinach Leaves"))
            .value(beverage -> beverage.getImage(), containsString(bucketUrl))
            .value(beverage -> beverage.getRecipe(), notNullValue());
  }

  @Test
  @DisplayName("Should get beverage by ingredient")
  void shouldGetBeverageByIngredientKeywordIgnoreCase() {
    webClient.get().uri("/beverages/ingredients/{keyword}", "pear")
            .exchange()
            .expectStatus().isOk()
            .expectHeader().contentType(APPLICATION_JSON_VALUE)
            .expectBodyList(Beverage.class)
            .value(beverages ->
                    beverages.forEach( beverage ->
                            assertTrue(beverage.getIngredients().toLowerCase().contains("pear"))));
  }

  @Test
  @DisplayName("Should get beverage by ingredient in capitalize")
  void shouldGetBeverageByIngredientKeyword() {
    webClient.get().uri("/beverages/ingredients/{keyword}", "Pear")
            .exchange()
            .expectStatus().isOk()
            .expectHeader().contentType(APPLICATION_JSON_VALUE)
            .expectBodyList(Beverage.class);
  }

}
```

In our test case `shouldGetBeverage()` we are validating beverage content:

* `equalTo` Validates equals between two values
* `containsString` Validates contains a string value
* `notNullValue` Validates is not null

In our test case `shouldGetBeverageByIngredientKeywordIgnoreCase()` we are validating that every beverage in the collection has pear in ingredients.

To browse the complete project go [here](https://github.com/josdem/jugoterapia-webflux), to download the project:

```bash
git clone git@github.com:josdem/jugoterapia-webflux.git
```

To run the project:

```bash
gradle bootRun -Dspring.data.mongodb.username=username -Dspring.data.mongodb.password=password
```

To test the project:

```bash
gradle test -Dspring.data.mongodb.username=username -Dspring.data.mongodb.password=password
```

[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
