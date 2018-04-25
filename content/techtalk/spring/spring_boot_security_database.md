+++
title = "Spring Boot Security using Database"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology", "Spring", "spring boot"]
date = "2017-01-21T20:55:53-06:00"
description = "This post walks you through the process of creating a simple registration and login example with Spring Security using database."
+++

This post walks you through the process of creating a simple registration and login example with Spring Security using database. Please read this previous [post](/techtalk/spring/spring_boot_security) before conitnue with this information. First we need to add Spring Data JPA dependency to the `build.gradle`

```groovy
compile 'org.springframework.boot:spring-boot-starter-data-jpa'
```

Then we need to change Spring security Java config class in order to add `userDetailsService` to implement database access functionality:

```groovy
package com.jos.dem.springboot.security.config

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.context.annotation.Configuration
import org.springframework.security.core.userdetails.UserDetailsService
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder
import org.springframework.security.config.annotation.web.builders.HttpSecurity
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity

@Configuration
@EnableWebSecurity
class SecurityConfig extends WebSecurityConfigurerAdapter {

  @Autowired
  UserDetailsService userDetailsService

  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http
    .authorizeRequests()
    .antMatchers("/assets/**").permitAll()
    .anyRequest().authenticated()
    .and()
    .formLogin()
    .loginPage("/login")
    .permitAll()
  }

  @Autowired
  void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
    auth
    .userDetailsService(userDetailsService)
    .passwordEncoder(new BCryptPasswordEncoder())
  }

}
```

This is the `userDetailsService` implementation

```groovy
package com.jos.dem.springboot.security.service.impl

import org.springframework.stereotype.Service
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.security.core.GrantedAuthority
import org.springframework.security.core.userdetails.UserDetailsService
import org.springframework.security.core.authority.SimpleGrantedAuthority
import org.springframework.security.core.userdetails.UsernameNotFoundException

import  com.jos.dem.springboot.security.model.User
import  com.jos.dem.springboot.security.repository.UserRepository

@Service
class UserDetailsServiceImpl implements UserDetailsService {

  @Autowired
  UserRepository userRepository

  @Override
  org.springframework.security.core.userdetails.User loadUserByUsername(String username) throws UsernameNotFoundException {
    User user = userRepostory.findByUsername(username)
    Set<GrantedAuthority> grantedAuthorities = [] as Set
    grantedAuthorities << new SimpleGrantedAuthority(user.role.toString())
    new org.springframework.security.core.userdetails.User(user.username, user.password, grantedAuthorities)
  }

}
```

JPA Entity is defined with `@Entity` annotation to represent a table in your database, this is the User entity to represent an user with credentials.

```groovy
package com.jos.dem.springboot.security.model

import static javax.persistence.EnumType.STRING
import static javax.persistence.GenerationType.AUTO

import javax.persistence.Id
import javax.persistence.Column
import javax.persistence.Entity
import javax.persistence.Enumerated
import javax.persistence.GeneratedValue

@Entity
class User {

  @Id
  @GeneratedValue(strategy=AUTO)
  Long id
  @Column(unique = true, nullable = false)
  String username
  @Column(nullable = false)
  String password
  @Column(nullable = true)
  String firstname
  @Column(nullable = true)
  String lastname
  @Column(nullable = true)
  String email
  @Column(nullable = false)
  @Enumerated(STRING)
  Role role
  @Column(nullable = false)
  Boolean enabled = false
  @Column(nullable = false)
  Date dateCreate = new Date()

}
```

Any user in the system has a Role

```groovy
package com.jos.dem.springboot.security.model

enum Role {
  USER,ADMIN
}
```

This is our `UserRepository` to find a user in a dabatabase by username and save a new user.

```groovy
package com.jos.dem.springboot.security.repository

import com.jos.dem.springboot.security.model.User
import org.springframework.data.jpa.repository.JpaRepository

interface UserRepository extends JpaRepository<User,Long> {

  User findByUsername(String username)
  User save(User user)

}
```

Finally we are going to create a default user in our database with encrypted password, in order to do that we are going to implement `ApplicationReadyEvent` in a Bootstrap class, that's it, when Spring Boot is up and running we are going to verify if a default user exist in our database otherwise is going to create it.

```groovy
package com.jos.dem.springboot.security.config

import org.springframework.stereotype.Component
import org.springframework.context.ApplicationListener
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.context.event.ApplicationReadyEvent
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder

import com.jos.dem.springboot.security.model.User
import com.jos.dem.springboot.security.model.Role
import com.jos.dem.springboot.security.repository.UserRepository

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@Component
class Boostrap implements ApplicationListener<ApplicationReadyEvent>{

  @Autowired
  UserRepository userRepository

  Logger log = LoggerFactory.getLogger(this.class)

  @Override
  void onApplicationEvent(final ApplicationReadyEvent event) {
    log.info 'Verifying if default user exist'
    createUserWithRole('josdem', '12345678', 'joseluis.delacruz@gmail.com', Role.USER)
  }

  private createUserWithRole(String username, String password, String email, Role authority){
    if(!userRepository.findByUsername(username)){
      User user = new User(
        username:username,
        password:new BCryptPasswordEncoder().encode(password),
        email:email,
        role:authority,
        firstname:username,
        lastname:username,
        enabled:true
      )
      userRepository.save(user)
    }
  }

}
```

Do not forget to create your `application.properties` file, so you can specify your local database credentials there.

```properties
spring.datasource.url=jdbc:mysql://localhost/spring_boot_security
spring.datasource.username=username
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

spring.jpa.generate-ddl=true

spring.messages.basename=i18n/messages
```

To download the project

```bash
git clone https://github.com/josdem/spring-boot-security.git
git fetch
git checkout feature/database
```

To run the project:

```bash
gradle bootRun
```


[Return to the main article](/techtalk/spring)

