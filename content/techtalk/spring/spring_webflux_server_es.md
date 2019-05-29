+++
title =  "Spring Webflux el lado del Servidor"
categories = ["techtalk", "code","spring webflux"]
tags = ["josdem", "techtalks","programming","technology", "Reactive Programming", "WebFlux", "Reactor", "Java","spring webflux"]
date = "2018-03-26T16:10:59-06:00"
description = "This post walks you through the process of creating endpoints using Spring Webflux routers."
+++

Este post técnico nos llevará a través del proceso de crear end-points usando Spring Webflux. Por favor lee mi previo post [Spring Webflux Basics](/techtalk/spring/spring_webflux_basics) antes de continuar con esta información. Vamos a usar `@RestController` para poder servir nuestros datos desde `PersonRepository`. Consideremos el siguiente ejemplo:

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

  @GetMapping("/persons/{nickname}")
  public Mono<Person> findById(@PathVariable String nickname){
    log.info("Calling find person by id: {}", nickname);
    return personRepository.findById(nickname);
  }

}
```

Spring Web MVC fue construido pensando en Servlet API y Servlet containers y el servidor de aplicaciones comúnmente usado era Tomcat. El framework reactivo Spring Webflux es pensado en ser no-bloqueante, soporado por reactive strams y que corra unsando Netty server. Cuando usamos `@RestController` indicamos que no queremos renderear vistas, pero escribir los resultados directo al cuerpo de la respuesta. `@GetMapping` es la forma corta de `@RequestMapping(method=RequestMethod.GET)`. Así, después de agregar este controller al proyecto y correr Spring Boot, tú puedes ir a: [http://localhost:8080/persons](http://localhost:8080/persons)

```json
[
   {
      "nickname":"mkheck",
      "email":"mkheck@email.com"
   },
   {
      "nickname":"josdem",
      "email":"joseluis.delacruz@gmail.com"
   },
   {
      "nickname":"edzero",
      "email":"edzero@email.com"
   },
   {
      "nickname":"siedrix",
      "email":"siedrix@email.com"
   },
   {
      "nickname":"tgrip",
      "email":"tgrip@email.com"
   }
}
```

Este es el endpoint para obtener una persona por id: [http://localhost:8080/persons/josdem](http://localhost:8080/persons/josdem)

```json
{
  "nickname": "josdem",
  "email": "josdem@email.com"
}
```

**Conclusión**

Concluiremos con lo siguiente:

* Spring Boot ahora fomenta la programación reactiva
* Usando el estilo MVC para los controladores es una buena idea para empezar a adoptar web reactivo
* Webflux combina bien con Java y la programación funcional
* Webflux corre sobre Netty server

Para correr este proyecto con Gradle:

```bash
gradle bootRun
```

Para correr este proyecto con Maven:

```bash
mvn spring-boot:run
```

Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/reactive-webflux-workshop), para descargar el proyecto:

```bash
git clone https://github.com/josdem/reactive-webflux-workshop.git
cd server
```

[Regresar al artículo principal](/techtalk/spring#Spring_Boot_Reactive)
