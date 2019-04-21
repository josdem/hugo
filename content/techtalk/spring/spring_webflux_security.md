+++
title =  "Spring Webflux Security"
categories = ["techtalk", "code","spring webflux"]
tags = ["josdem", "techtalks","programming","technology", "webflux", "spring security","spring webflux"]
date = "2018-04-10T09:46:11-05:00"
description = "Spring Security is a powerful and highly customizable authentication and access-control framework. In this example I will show you how to integrate it to your Spring Reactive Webflux project."
+++

Spring Security is a powerful and highly customizable authentication and access-control framework. In this example I will show you how to integrate it to your Spring Reactive Webflux project.

Let's start creating a new Spring Boot project with Webflux, Security and Thymeleaf as dependencies:

```bash
spring init --dependencies=webflux,security,thymeleaf --build=gradle --language=java reactive-webflux-security
```

Here is the complete `build.gradle` file generated:

```java
buildscript {
	ext {
		springBootVersion = '2.0.1.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.security'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}

dependencies {
	compile('org.springframework.boot:spring-boot-starter-webflux')
	compile('org.springframework.boot:spring-boot-starter-security')
  compile('org.springframework.boot:spring-boot-starter-thymeleaf')
	testCompile('org.springframework.boot:spring-boot-starter-test')
	testCompile('io.projectreactor:reactor-test')
}
```

Spring Security’s `@EnableWebFluxSecurity` annotation enable WebFlux support in Spring Security. `SecurityWebFilterChain` provides some convenient defaults to get our application up and running quickly. A small improvement in Spring Security is a new styled login form which uses the Bootstrap 4 CSS framework, in order to use the new login form, let’s add the corresponding `formLogin()` builder method to the `ServerHttpSecurity`.

```java
package com.jos.dem.security.config;

import org.springframework.context.annotation.Bean;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.MapReactiveUserDetailsService;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.config.annotation.web.reactive.EnableWebFluxSecurity;
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
  public MapReactiveUserDetailsService userDetailsRepository() {
    UserDetails user = User.withDefaultPasswordEncoder()
      .username("josdem")
      .password("12345678")
      .roles("USER")
      .build();
    return new MapReactiveUserDetailsService(user);
  }

}
```

`userDetailsRepository` provides us with a convenient mock user builder and an in-memory implementation of the user details service.

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

In the Controller we can see `Principal` and it can be defined directly as a method argument and it will be correctly resolved by the framework. `Principal` is the currently logged in user. This interface represents the abstract notion of a principal, which can be used to represent any entity, such as an individual or corporation.

```html
<html>
  <body>
    <p th:inline="html">
       Hello, [[${username}]]!
    </p>
  </body>
</html>
```

Now if we run the project `gradle bootRun` and we now open the main page of the application: [http://localhost:8080](http://localhost:8080) we’ll see that it looks much better than the default form we’re used to since previous versions of Spring Security.

<img src="/img/techtalks/spring/login_form.png">

After a successful login you can see a greeting

<img src="/img/techtalks/spring/form_greeting.png">

To browse the project go [here](https://github.com/josdem/reactive-webflux-security), to download the project:

```bash
git clone https://github.com/josdem/reactive-webflux-security.git
git fetch
git checkout feature/in-memory
```

To run the project:

```bash
gradle bootRun
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
