+++
date = "2017-01-10T19:25:41-06:00"
title = "Spring Boot Security"
tags = ["josdem","techtalks","programming","technology","spring boot","Custom Login Form Spring Boot"]
categories = ["techtalk","code"]

+++

Spring Security is a powerful and highly customizable authentication and access-control framework. In this example I will show you how to integrate it to your Spring Boot project. First you set up a basic build script.`build.gradle`

```groovy
buildscript {
  ext {
    springBootVersion = '2.0.0.RELEASE'
  }
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  }
}

apply plugin: 'groovy'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.security'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
  mavenCentral()
}

dependencies {
  compile('org.springframework.boot:spring-boot-starter-web')
  compile('org.springframework.boot:spring-boot-starter-security')
  compile('org.springframework.boot:spring-boot-starter-thymeleaf')
  compile('org.codehaus.groovy:groovy-all')
  testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

Spring Security’s `WebSecurityConfigurerAdapter` provides some convenient defaults to get our application up and running quickly. Let’s see how we can update our configuration to use a custom form.

```groovy
package com.jos.dem.springboot.security.config

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.context.annotation.Configuration
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder
import org.springframework.security.config.annotation.web.builders.HttpSecurity
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity

@Configuration
@EnableWebSecurity
class SecurityConfig extends WebSecurityConfigurerAdapter {

  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http
    .authorizeRequests()
    .antMatchers("/assets/**").permitAll() // 1
    .anyRequest().authenticated()          // 2
    .and()
    .formLogin()                           // 3
    .loginPage("/login")                   // 4
    .permitAll()
  }

  @Autowired
  void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
    auth
    .inMemoryAuthentication()
    .withUser("josdem").password("12345678").roles("USER")
  }

}
```

1. We grant full access to the assets folder (JavaScript and CSS)
2. Every request requires the user to be authenticated
3. Form based authentication is supported
4. Login request mapping

**Besides**

The `SecurityConfig` class is annotated with `@EnableWebSecurity` to enable Spring Security’s web security support and provide the Spring MVC integration.

The `configure(HttpSecurity)` method defines which URL paths should be secured and which should not.

The `configureGlobal(AuthenticationManagerBuilder)` method, it sets up an in-memory user store with a single user. That user is given a username of `"user"`, a password of `"12345678"`, and a role of `"USER"`.

Now we need to create the login page:

```html
<html>
  <head>
    <link rel="stylesheet" th:href="@{/assets/third-party/bootstrap/dist/css/bootstrap.min.css}" />
  </head>
  <body>
    <nav class="navbar navbar-inverse navbar-fixed-top">
      <font color="white"><h3 th:text="#{login.header}"/></font>
    </nav>
    <br/><br/><br/><br/>
    <div class="container">
      <form th:action="@{/login}" method='POST' id='loginForm' class='cssform' autocomplete='off'>
        <div class="form-group">
          <label for="username"><h4 th:text="#{login.username}"/></label>
          <input type="text" name='username' class="form-control" placeholder="username" id='username'/>
        </div>
        <div class="form-group">
          <label for="password"><h4 th:text="#{login.password}"/></label>
          <input type="password" name='password' class="form-control" placeholder="password" id='password'/>
        </div>
        <br/>
        <button id="btn-success" type="submit" class="btn btn-lg btn-primary btn-block"><h5 th:text="#{login.action}"/></button>
        <br/>
      </form>
    </div>
    <footer>
      <nav class="navbar navbar-inverse navbar-fixed-bottom">
        <a class="navbar-brand" th:href="#{project.repository}"><p th:text="#{login.footer}"/></a>
      </nav>
    </footer>
  </body>
</html>
```

This is our `LoginController` we are rendering error object message in case that the attempt to login failed and the view is specified.


```groovy
package com.jos.dem.springboot.security.controller

import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RequestParam
import org.springframework.web.servlet.ModelAndView
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.beans.factory.annotation.Value
import org.springframework.stereotype.Controller

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@Controller
class LoginController {

  Logger log = LoggerFactory.getLogger(this.class)

  @RequestMapping("/login")
  ModelAndView login(@RequestParam Optional<String> error){
    log.info "Calling login"
    ModelAndView modelAndView = new ModelAndView('login/login')
    if(error.isPresent()){
      modelAndView.addObject('error', 'Invalid Credentials')
    }
    modelAndView
  }

}
```

In Spring Boot 2.0.0.RELEASE we need to specify `PasswordEncoder` therefore we need to add that bean to the configuration. I am attaching here our `SpringBootApplication` class

```groovy
package com.jos.dem.springboot.security

import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.context.annotation.Bean
import org.springframework.security.crypto.password.PasswordEncoder
import org.springframework.security.crypto.password.NoOpPasswordEncoder

@SpringBootApplication
class DemoApplication {

  @Bean
  PasswordEncoder passwordEncoder() {
    NoOpPasswordEncoder.getInstance()
  }

  static void main(String[] args) {
    SpringApplication.run DemoApplication, args
  }
}
```

Finally this is our `HomeController` with a protected message:

```groovy
package com.jos.dem.springboot.security.controller

import org.springframework.stereotype.Controller
import org.springframework.web.bind.annotation.RequestMapping

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@Controller
class HomeController {

  Logger log = LoggerFactory.getLogger(this.class)

  @RequestMapping('/')
  String index(){
    log.info 'Calling Index'
    'index'
  }

}
```

A protected messsage is just a `Hello World!` page

```html
<html>
  Hello World!
</html>
```

Now run the project and try visiting http://localhost:8080/ to see a Hello World message previous authentication using our custom login page.

To download the project

```bash
git clone https://github.com/josdem/spring-boot-security.git
```

To run the project:

```bash
gradle bootRun
```

[Return to the main article](/techtalk/spring)

