+++
title =  "Spring Webflux Security"
categories = ["techtalk", "code","spring webflux"]
tags = ["josdem", "techtalks","programming","technology", "webflux", "spring security","spring webflux"]
date = "2018-04-10T09:46:11-05:00"
description = "Spring Security is a powerful and highly customizable authentication and access-control framework. In this example I will show you how to integrate it to your Spring Reactive Webflux project."
+++

Spring Security is a powerful and highly customizable authentication and access-control framework. In this technical post we will go through the process to integrate Spring Security to Spring Reactive Webflux. If you want to know more about how to create Spring Webflux please go to my previous post getting started with Spring Webflux [here](/techtalk/spring/spring_webflux_basics). Let's begin creating a new Spring Boot project with Webflux, Lombok, Spring Security and Thymeleaf:

```bash
spring init --dependencies=webflux,lombok,security,thymeleaf --build=gradle --language=java reactive-webflux-security
```

Here is the complete `build.gradle` file generated:

```java
plugins {
  id 'org.springframework.boot' version '2.5.4'
  id 'io.spring.dependency-management' version '1.0.11.RELEASE'
  id 'java'
}

group = 'com.jos.dem.security'
version = '1.0.0-SNAPSHOT'
sourceCompatibility = 16

configurations {
  compileOnly {
    extendsFrom annotationProcessor
  }
}

repositories {
  mavenCentral()
}

dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-security'
  implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
  implementation 'org.springframework.boot:spring-boot-starter-webflux'
  implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity5'
  compileOnly 'org.projectlombok:lombok'
  annotationProcessor 'org.projectlombok:lombok'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
  testImplementation 'io.projectreactor:reactor-test'
  testImplementation 'org.springframework.security:spring-security-test'
  testCompileOnly 'org.projectlombok:lombok'
  testAnnotationProcessor 'org.projectlombok:lombok'
}

test {
  useJUnitPlatform()
}
```

Spring Security’s `@EnableWebFluxSecurity` annotation enable WebFlux support in Spring Security. `SecurityWebFilterChain` provides some convenient defaults to get our application up and running quickly. A small improvement in Spring Security is a new styled login form which uses the Bootstrap 4 CSS framework, in order to use the new login form, let’s add the corresponding `formLogin()` builder method to the `ServerHttpSecurity`.

```java
package com.jos.dem.security.config;

import com.jos.dem.security.model.Roles;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.security.config.annotation.web.reactive.EnableWebFluxSecurity;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.core.userdetails.MapReactiveUserDetailsService;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.server.SecurityWebFilterChain;

@EnableWebFluxSecurity
@RequiredArgsConstructor
public class SecurityConfig {

  private final ApplicationConfig config;

  @Bean
  public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
    http.authorizeExchange().anyExchange().authenticated().and().httpBasic().and().formLogin();
    return http.build();
  }

  @SuppressWarnings("deprecation")
  @Bean
  public MapReactiveUserDetailsService userDetailsService() {
    UserDetails user =
        User.withDefaultPasswordEncoder()
            .username(config.getUsername())
            .password(config.getPassword())
            .roles(Roles.USER.name())
            .build();
    return new MapReactiveUserDetailsService(user);
  }
}
```

`userDetailsService` provides a convenient mock user builder and an in-memory implementation from user details.

```java
package com.jos.dem.security.controller;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

import java.security.Principal;

@Slf4j
@Controller
@RequiredArgsConstructor
public class SecurityController {

  @GetMapping("/")
  public String index(Model model, Principal principal) {
    String username = principal.getName();
    log.info("Authenticated user is: " + username);
    model.addAttribute("username", username);
    return "home";
  }
}
```

In this controller we can see `Principal` and it can be defined directly as a method argument and it will be correctly resolved by the framework. `Principal` is the currently logged in user. This interface represents the abstract notion of a principal, which can be used to represent any entity, whether individual or corporation.

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

Finally here is our main Spring Boot demo application

```java
package com.jos.dem.security;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class SecurityApplication {

  public static void main(String[] args) {
    SpringApplication.run(SecurityApplication.class, args);
  }
}
```

To run the project:

```bash
gradle bootRun
```

**Using Maven**

You can do the same using Maven, the only difference is that you need to specify `--build=maven` parameter in the `spring init` command line:

```bash
spring init --dependencies=webflux,lombok,security,thymeleaf --build=maven --language=java reactive-webflux-security
```

And when you run your project use this command:

```bash
mvn spring-boot:run
```

This is the pom.xml file generated:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.4</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <groupId>com.jos.dem.webflux</groupId>
  <artifactId>reactive-webflux-workshop</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>demo</name>
  <description>Demo project for Spring Webflux Security</description>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>16</java.version>
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
      <groupId>org.thymeleaf.extras</groupId>
      <artifactId>thymeleaf-extras-springsecurity5</artifactId>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
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
        <configuration>
          <excludes>
            <exclude>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
            </exclude>
          </excludes>
        </configuration>
      </plugin>
    </plugins>
  </build>

</project>
```

To browse the project go [here](https://github.com/josdem/reactive-webflux-security), to download the project:

```bash
git clone https://github.com/josdem/reactive-webflux-security.git
git fetch
git checkout in-memory
```



[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
