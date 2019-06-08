+++
title =  "Spring Webflux Security Database"
categories = ["techtalk", "code","spring webflux"]
tags = ["josdem", "techtalks","programming","technology","Spring Security", "Webflux Security","spring webflux"]
date = "2018-04-11T16:12:30-05:00"
description = "This post walks you through the process of creating a simple registration and login example with Spring WebFlux Security using MongoDB database."
+++

This post walks you through the process of creating a simple registration and login example with Spring WebFlux Security using MongoDB database. Please read this previous [Spring Webflix Security](/techtalk/spring/spring_webflux_security) before conitnue with this information. Let's add MongoDB and Lombok dependencies to the `build.gradle` file

```groovy
implementation('org.springframework.boot:spring-boot-starter-data-mongodb-reactive')
compileOnly('org.projectlombok:lombok')
annotationProcessor('org.projectlombok:lombok')
```

Lombok is a great tool to avoid boilerplate code, for knowing more please go [here](https://projectlombok.org/). Next step,  we need to change Spring security Java config class in order to add `userDetailsService`, so that we can implement database access functionality.

```java
package com.jos.dem.security.config;

import org.springframework.context.annotation.Bean;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.ReactiveUserDetailsService;
import org.springframework.security.config.annotation.web.reactive.EnableWebFluxSecurity;
import org.springframework.security.config.web.server.ServerHttpSecurity;
import org.springframework.security.web.server.SecurityWebFilterChain;

import com.jos.dem.security.repository.UserRepository;

@EnableWebFluxSecurity
public class SecurityConfig {

  @Autowired
  private UserRepository userRepository;

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
  public ReactiveUserDetailsService  userDetailsService() {
    return (username) -> userRepository.findByUsername(username);
  }

}
```

Since we’re in reactive land, the user details service should also be reactive. So let's usee a `ReactiveMongoRepository` that returns a Mono publisher.

```java
package com.jos.dem.security.repository;

import com.jos.dem.security.model.User;

import org.springframework.data.mongodb.repository.ReactiveMongoRepository;
import org.springframework.security.core.userdetails.UserDetails;

import reactor.core.publisher.Mono;

public interface UserRepository extends ReactiveMongoRepository<User, String> {
  Mono<UserDetails> findByUsername(String username);
}
```

Here is our user class that implements `UserDetails` specification.

```java
package com.jos.dem.security.model;

import java.util.Collection;
import java.util.HashSet;
import java.util.Set;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import lombok.Builder;
import lombok.Data;
import lombok.ToString;

@Data
@Document
@ToString
public class User implements UserDetails {

  @Id
  private String username;
  private String password;

  private boolean active = true;
  private Set<GrantedAuthority> roles = new HashSet<GrantedAuthority>();

  @Builder
  public User(String username, String password){
    this.username = username;
    this.password = password;
    roles.add(new SimpleGrantedAuthority("ROLE_USER"));
  }

  @Override
  public Collection<? extends GrantedAuthority> getAuthorities() {
    return roles;
  }

  @Override
  public boolean isAccountNonExpired() {
    return active;
  }

  @Override
  public boolean isAccountNonLocked() {
    return active;
  }

  @Override
  public boolean isCredentialsNonExpired() {
    return active;
  }

  @Override
  public boolean isEnabled() {
    return active;
  }

  @Override
  public String getPassword() {
    return this.password;
  }

  @Override
  public String getUsername() {
    return this.username;
  }

}
```

Now, we are going to use `CommandLineRunner` to save an user. `CommandLineRunner` is a call back interface in Spring Boot, so when Spring Boot starts will call it and pass in args through a `run()` internal method.

```java
package com.jos.dem.security;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.crypto.factory.PasswordEncoderFactories;

import com.jos.dem.security.model.User;
import com.jos.dem.security.repository.UserRepository;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@SpringBootApplication
public class DemoApplication {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }

  @Bean
  PasswordEncoder passwordEncoder() {
    return PasswordEncoderFactories.createDelegatingPasswordEncoder();
  }

  @Bean
  CommandLineRunner start(UserRepository userRepository, PasswordEncoder passwordEncoder){
    return args -> {
      User user = new User("josdem", passwordEncoder.encode("12345678"));
      userRepository.save(user).subscribe();

      userRepository.findAll().log().subscribe(u -> log.info("user: {}", u));
    };
  }

}
```

Do not forget to create a Mongo database in your local environment and specify your credentials in the `application.properties` file:

```properties
spring.data.mongodb.database=webflux_security
spring.data.mongodb.host=localhost
spring.data.mongodb.username=username
spring.data.mongodb.password=password
```

Now we are good to go and start our application using `gradle bootRun` command and then list our user collection from MongoDB.

```bash
2019-06-08 14:49:18.694 INFO - [ntLoopGroup-2-3] application$$EnhancerBySpringCGLIB$$33cdb075 : user: User(uuid=5cfc029c5061f8554e0b6e95, username=josdem, password={bcrypt}$2a$10$ucxaZC48cuVSWxoKfvrr7.PEOGkX1iR9mXWXQey5vECfiXF/bewTm, active=true, roles=[ROLE_USER])
```

Now if we open the main page of the application: [http://localhost:8080](http://localhost:8080) we’ll see a nice and good looking Bootstrap 4 form to accept our user.

<img src="/img/techtalks/spring/login_form.png">

After a successful login you can see a greeting

<img src="/img/techtalks/spring/form_greeting.png">


To browse the project go [here](https://github.com/josdem/reactive-webflux-security), to download the project:

```bash
git clone https://github.com/josdem/reactive-webflux-security.git
git fetch
git checkout database
```

To run the project:

```bash
gradle bootRun
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
