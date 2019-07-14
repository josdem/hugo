+++
title =  "Seguridad con Spring Webflux"
categories = ["techtalk", "code","spring webflux"]
tags = ["josdem", "techtalks","programming","technology", "webflux", "spring security","spring webflux"]
date = "2018-04-10T09:46:11-05:00"
description="Spring Security es un poderoso y altamente customizable framework de control de acceso. En este ejemplo veremos como integrar Spring Security en una aplicación reactiva Spring Webflux."
+++

Spring Security es un poderoso y altamente customizable framework de control de acceso. En este ejemplo veremos como integrar Spring Security en una aplicación reactiva Spring Webflux. Si quieres saber más acerca de como crear una aplicación Spring Webflux por favor ve a mi previo post técnico [Empezando con Spring Webflux](/techtalk/spring/spring_webflux_basics_es). Vamos a empezar creando un nuevo proyecto Spring Boot con Webflux, Security y Thymeleaf como dependencias:

```bash
spring init --dependencies=webflux,security,thymeleaf --build=gradle --language=java reactive-webflux-security
```

Aquí esta el `build.gradle` generado:

```java
plugins {
  id 'org.springframework.boot' version '2.1.5.RELEASE'
  id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.security'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 11

repositories {
  mavenCentral()
}

dependencies {
  implementation('org.springframework.boot:spring-boot-starter-webflux')
  implementation('org.springframework.boot:spring-boot-starter-security')
  implementation('org.springframework.boot:spring-boot-starter-thymeleaf')
  testImplementation('org.springframework.boot:spring-boot-starter-test')
  testImplementation('io.projectreactor:reactor-test')
}
```

La anotación Spring Security `@EnableWebFluxSecurity` habilita el soporte WebFlux en Spring Security. `SecurityWebFilterChain` provee confifguración por default para poder tener nuestra aplicación lista sin mucho esfuerzo. Una mejora en Spring Security es el formulario para el login el cual usa Bootstrap 4 CSS, ahora para poder utilizar este formulario vamos a agregar el correspondiente `formLogin()` al método builder de `ServerHttpSecurity`.


```java
package com.jos.dem.security.config;

import org.springframework.context.annotation.Bean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.MapReactiveUserDetailsService;
import org.springframework.security.config.annotation.web.reactive.EnableWebFluxSecurity;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.web.server.SecurityWebFilterChain;

@EnableWebFluxSecurity
public class SecurityConfig {

  @Bean
  public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
    http
      .authorizeExchange()
      .anyExchange()
      .authenticated()
      .and()
      .httpBasic()
      .and()
      .formLogin();
    return http.build();
  }

  @Bean
  public MapReactiveUserDetailsService userDetailsService() {
    UserDetails user = User.withDefaultPasswordEncoder()
      .username("josdem")
      .password("12345678")
      .roles("USER")
      .build();
    return new MapReactiveUserDetailsService(user);
  }

}
```

`userDetailsService` provee un conveniente método para construir un nuevo usuario en memoria.

```java
package com.jos.dem.security.controller;

import org.springframework.ui.Model;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

import java.security.Principal;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Controller
public class DemoController {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @GetMapping("/")
  public String index(Model model, Principal principal) {
    String username = principal.getName();
    log.info("Authenticated user is: " + username);
    model.addAttribute("username", username);
    return "home";
  }

}
```

En este controller podemos ver el objeto `Principal` y puede ser definido directamente como un argumento y así ser manejado correctamente por el framework. `Principal` es el usuario que se a logeado a la aplicación.

```html
<html>
  <body>
    <p th:inline="html">
       Hello, [[${username}]]!
    </p>
  </body>
</html>
```

Ahora si iniciamos el proyecto con `gradle bootRun` y abrimos la página principal de la aplicación: [http://localhost:8080](http://localhost:8080) podremos ver que luce mucho mejor que las versiones previas de Spring Security.

<img src="/img/techtalks/spring/login_form.png">

Después de un acceso exitoso puedes ver este saludo.

<img src="/img/techtalks/spring/form_greeting.png">

Finalmente aquí está nuestra aplicación main de Spring Boot.

```java
package com.jos.dem.security;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }

}
```

Para correr el proyecto:

```bash
gradle bootRun
```

**Usando Maven**

Tú puedes hacer lo mismo usando Maven, la única diferencia es que tienes que específicar el parámetro `--build=maven` en el comando `spring init`:

```bash
spring init --dependencies=webflux,security,thymeleaf --build=maven --language=java reactive-webflux-security
```

Entonces puedes correr el proyecto usando este comando:

```bash
mvn spring-boot:run
```

Este es el `pom.xml` generado:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem.webflux</groupId>
  <artifactId>reactive-webflux-workshop</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>demo</name>
  <description>Demo project for Spring Webflux Security</description>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.5.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>11</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>io.projectreactor</groupId>
      <artifactId>reactor-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework.security</groupId>
      <artifactId>spring-security-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>

</project>
```

Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/reactive-webflux-security), para descargar el proyecto:

```bash
git clone https://github.com/josdem/reactive-webflux-security.git
git fetch
git checkout in-memory
```


[Regresar al artículo principal](/techtalk/spring#Spring_Boot_Reactive_ES)
