+++
title =  "Seguridad con Spring Webflux y MongoDB"
categories = ["techtalk", "code","spring webflux"]
tags = ["josdem", "techtalks","programming","technology","Spring Security", "Webflux Security","spring webflux"]
date = "2018-04-11T16:12:30-05:00"
description = "This post walks you through the process of creating a simple registration and login example with Spring WebFlux Security using MongoDB database."
+++

Este post nos llevará a través del proceso de crear una aplicación de registro y login usando Spring Webflux Security y MongoDB. Por favor lee mi previo post [Spring Webflix Security](/techtalk/spring/spring_webflux_security_es) antes de continuar con esta información. Vamos a empezar agregando las dependencias de MongoDB y Lombok al archivo `build.gradle`

```groovy
implementation('org.springframework.boot:spring-boot-starter-data-mongodb-reactive')
compileOnly('org.projectlombok:lombok')
annotationProcessor('org.projectlombok:lombok')
```

Lombok es una gran herramienta para ahorrarnos código, para saber más acerca de Lombok por favor ve [aquí](https://projectlombok.org/). Ahora, necesitamos cambiar la configuración de Spring Security específicamente el método `userDetailsService`, así podemos obtener un usuario de la base de datos MongoDB.

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

Desde que estamos en el campo reactivo, `userDetailsService()` debería ser reactivo. Así que usemos `ReactiveMongoRepository` que regresa un publisher de tipo Mono.

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

Aquí está nuestra clase user que implementa la especificación `UserDetails`.

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

Ahora, vamos a usar `CommandLineRunner` para registrar un usuario. El `CommandLineRunner` es una interfaz call back en Spring Boot, cuando nuestra aplicación arranca llamará al método start y le pasará los argumentos unsando el método interno `run()`

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

No olvides crear la base de datos Mongo en tu ambiente local y especificar las credenciales en el archivo `application.properties`.

```properties
spring.data.mongodb.database=webflux_security
spring.data.mongodb.host=localhost
spring.data.mongodb.username=username
spring.data.mongodb.password=password
```

Ahora ya estamos listos para iniciar nuestra aplicación con el comando `gradle bootRun` y así poder listar nuestra colección de usuarios desde MongoDB.

```bash
2019-06-08 15:12:44.026  INFO 91949 - [ntLoopGroup-2-3] : user: User(username=josdem, password={bcrypt}$2a$10$Fyo6YP2SRe5MhOeQPD67KOoCIosizAsqcz98FZLvW0O2GFz10ag0a, active=true, roles=[ROLE_USER])
```

Finalmente abramos nuestra página principal de la aplicación: [http://localhost:8080](http://localhost:8080) y veremos un lindo y atractivo formulario con Bootstrap 4 para poder hacer login de nuestro usuario.

<img src="/img/techtalks/spring/login_form.png">

Después de el acceso exitoso, podrás ver este saludo.

<img src="/img/techtalks/spring/form_greeting.png">

**Usando Maven**

Para el soporte Maven agrega las dependencias de MongoDB y Lombok al archivo `pom.xml`

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
</dependency>
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <scope>provided</scope>
</dependency>
```

Para ejecutar el projecto con Gradle:

```bash
gradle bootRun
```

Para ejecutar el projecto con Maven:

```bash
mvn spring-boot:run
```

Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/reactive-webflux-security), para descargar el proyecto:

```bash
git clone https://github.com/josdem/reactive-webflux-security.git
git fetch
git checkout database
```



[Return to the main article](/techtalk/spring#Spring_Boot_Reactive_es)
