+++
title = "Spring Boot Security"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology","spring boot","Custom Login Form Spring Boot"]
date = "2017-01-10T19:25:41-06:00"
description = "Spring Security is a powerful and highly customizable authentication and access-control framework. In this example I will show you how to integrate it to your Spring Boot project."
+++

Spring Security is a powerful and highly customizable authentication and access-control framework. This technical post will review how to integrate security into your Spring Boot project. **NOTE:** If you need to know what tools you need to have installed on your computer to create a Spring Boot basic project, please refer to my previous post: [Spring Boot](/techtalk/spring_boot). Then, let’s create a new Spring Boot project with web, security and Lombok as dependencies:

```bash
spring init --dependencies=web,security,thymeleaf --build=gradle --type=gradle-project --language=java spring-boot-security
```

Here is the complete `build.gradle` file generated:

```groovy
plugins {
  id 'java'
  id 'org.springframework.boot' version '3.1.3'
  id 'io.spring.dependency-management' version '1.1.3'
}

group = 'com.josdem.springboot.security'
version = '0.0.1-SNAPSHOT'

java {
  sourceCompatibility = '17'
}

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
  implementation 'org.springframework.boot:spring-boot-starter-web'
  compileOnly 'org.projectlombok:lombok'
  annotationProcessor 'org.projectlombok:lombok'
  implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity6'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
  testImplementation 'org.springframework.security:spring-security-test'
  testCompileOnly 'org.projectlombok:lombok'
  testAnnotationProcessor 'org.projectlombok:lombok'
}

tasks.named('test') {
  useJUnitPlatform()
}
```

In the latest versions of Spring Security you will need to specify your view template names; here we have an example of how to do it.

```java
package com.josdem.springboot.security.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class MvcConfig implements WebMvcConfigurer {

    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/home").setViewName("home");
        registry.addViewController("/hello").setViewName("hello");
        registry.addViewController("/login").setViewName("login");
        registry.addViewController("/logout").setViewName("logout");
    }

}
```

Now we need to create the login page as: `${PROJECT_HOME}/src/main/resources/templates/login.html`:

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org">
<head>
    <title>Spring Security Example </title>
</head>
<body>
<div th:if="${param.error}">
    Invalid username and password.
</div>
<div th:if="${param.logout}">
    You have been logged out.
</div>
<form th:action="@{/login}" method="post">
    <div><label> Username : <input type="text" name="username"/> </label></div>
    <div><label> Password: <input type="password" name="password"/> </label></div>
    <div><input type="submit" value="Sign In"/></div>
</form>
</body>
</html>
```

This is our `HomeController`.


```java
package com.josdem.springboot.security.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Slf4j
@Controller
public class HomeController {

  @GetMapping("/")
  public String index() {
    log.info("Calling home");
    return "home";
  }
}
```

And home view:

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="https://www.thymeleaf.org">
<head>
  <title>Home Page</title>
</head>
<body>
<h1>Welcome!</h1>
<p>Click <a th:href="@{/message}">here</a> to see a private message.</p>
</body>
</html>
```

This home view includes a link to `/message` page, defined in the following [Thymeleaf](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html) template.

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head><title>Hello Page</title></head>
<body><h1>Hello World!</h1></body>
</html>
```

The expected behavior is when a user clicks on the `/message` link it will ask for credentials, in order to do that we need to define our `securityFilterChain` and `userDetailsService`

```java
package com.josdem.springboot.security.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class WebSecurityConfig {

  @Bean
  public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    http
            .authorizeHttpRequests((requests) -> requests
                    .requestMatchers("/", "/home").permitAll()
                    .anyRequest().authenticated()
            )
            .formLogin((form) -> form
                    .loginPage("/login")
                    .permitAll()
            )
            .logout((logout) -> logout.permitAll());

    return http.build();
  }

  @Bean
  public UserDetailsService userDetailsService() {
    UserDetails user =
            User.withDefaultPasswordEncoder()
                    .username("josdem")
                    .password("12345678")
                    .roles("USER")
                    .build();

    return new InMemoryUserDetailsManager(user);
  }

}
```

Finally this is our `MessageController`:

```java
package com.josdem.springboot.security.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Slf4j
@Controller
public class MessageController {

    @GetMapping("/message")
    public String message() {
        log.info("Calling private message");
        return "message";
    }
}
```

Now run the project:

```bash
gradle bootRun
```

To browse the project go [here](https://github.com/josdem/spring-boot-security), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-security.git
git fetch
git checkout feature/in-memory
```

[Return to the main article](/techtalk/spring#Spring_Boot)
