+++
title =  "Spring Webflux Testeando la Capa Web"
description = "Spring webflux web testing"
date = "2019-02-03T13:00:31-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

En este post técnico iremos a través del proceso de testear una aplicación web reactiva usando [WebTestClient](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/test/web/reactive/server/WebTestClient.html). WebTestClient nos ayuda a testear controladores Spring [WebFlux](https://docs.spring.io/spring/docs/current/spring-framework-reference/web-reactive.html) con auto configuración, si quieres saber más acerca de como crear una aplicación Spring Webflux por favor ve a mi previo post previo empezando con Spring Webflux [aquí](/techtalk/spring/spring_webflux_basics). Como proyecto ejemplo vamos usar [Jugoterapia WebFlux](https://github.com/josdem/jugoterapia-webflux) el cual provee recetas saludables de jugos y licuados. En este post técnico veremos como testear los controladores de ese proyecto. Ahora por favor considera este primer controlador.

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

La responsabilidad de este controlador es proveer un mecanismo de verificar que el servicio está "up and running!", este es el test case.

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

Desde que Jugoterapia WebFlux es una aplicación Spring Boot estamos usando la anotación `@SpringBootTest` la cual específica que testeará una aplicacición de ese tipo, también estamos usando `WebEnvironment` el cual crea una aplicación de conexto reactiva escuchando en un puerto aleatorio.

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

Este controlador obtiene las categorías por lenguaje y por id y aquí esta el test case:

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

Finalmente tenemos el controlador que provee las bebidas por id

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

Aquí está el test case:

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

Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/jugoterapia-webflux), para descargar el proyecto:

```bash
git clone git@github.com:josdem/jugoterapia-webflux.git
```

Para ejecutar el proyecto:

```bash
gradle bootRun
```

Para ejecutar el proyecto:

```bash
gradle test
```

[Return to the main article](/techtalk/spring#Spring_Boot_Reactive_es)
