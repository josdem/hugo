+++
tags = ["josdem","techtalks","programming","technology", "Spring"]
date = "2017-01-21T20:55:53-06:00"
title = "Spring Boot Security using Database"
categories = ["techtalk","code"]

+++

This post walks you through the process of creating a simple registration and login example with Spring Security using database. Please read this previous [post](/techtalk/spring/spring_boot_security) before conitnue with this information. First we need to add Spring Data JPA dependency to the build.gradle

```groovy
compile 'org.springframework.boot:spring-boot-starter-data-jpa'
```

Then we need to change Spring security Java config class in order to add `userDetailsService` to implement database access functionality:

```groovy
package com.jos.dem.vetlog.config

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.context.annotation.Configuration
import org.springframework.security.core.userdetails.UserDetailsService
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder
import org.springframework.security.config.annotation.web.builders.HttpSecurity
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder


@Configuration
@EnableWebSecurity
class SecurityConfig extends WebSecurityConfigurerAdapter {

  @Autowired
  UserDetailsService userDetailsService

  @Override
  void configure(HttpSecurity http) throws Exception {
    http
    .authorizeRequests()
    .antMatchers("/", "/assets/**","/home/**","/user/**").permitAll()
    .anyRequest().authenticated()
    .and()
    .formLogin()
    .loginPage("/login")
    .usernameParameter("username")
    .permitAll()
    .and()
    .logout()
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
package com.jos.dem.vetlog.service.impl

import org.springframework.stereotype.Service
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.security.core.GrantedAuthority
import org.springframework.security.core.userdetails.UserDetailsService
import org.springframework.security.core.authority.SimpleGrantedAuthority
import org.springframework.security.core.userdetails.UsernameNotFoundException

import  com.jos.dem.vetlog.model.User
import  com.jos.dem.vetlog.model.Role
import  com.jos.dem.vetlog.service.UserService

@Service
class UserDetailServiceImpl implements UserDetailsService {

  @Autowired
  UserService userService

  @Override
  org.springframework.security.core.userdetails.User loadUserByUsername(String username) throws UsernameNotFoundException {
    User user = userService.getUserByUsername(username)
    Set<GrantedAuthority> grantedAuthorities = new HashSet<>()
    grantedAuthorities.add(new SimpleGrantedAuthority(user.role.toString()))
    new org.springframework.security.core.userdetails.User(user.username, user.password, grantedAuthorities)
  }

}
```

JPA Entity is defined with `@Entity` annotation to represent a table in your database, this is the User entity to represent an user with credentials.

```groovy
package com.jos.dem.vetlog.model

import static javax.persistence.EnumType.STRING
import static javax.persistence.GenerationType.AUTO

import java.io.Serializable
import javax.persistence.Id
import javax.persistence.Column
import javax.persistence.Entity
import javax.persistence.Enumerated
import javax.persistence.GeneratedValue

@Entity
class User implements Serializable {

  @Id
  @GeneratedValue(strategy=AUTO)
  Long id
  @Column(unique = true, nullable = false)
  String username
  @Column(nullable = false)
  String password
  @Column(nullable = true)
  String firstName
  @Column(nullable = true)
  String lastName
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
package com.jos.dem.vetlog.model

enum Role {
  USER,ADMIN
}
```

That's it, when the controller call to save a new user, we will call this `userService` implementation:

```groovy
package com.jos.dem.vetlog.service.impl

import org.springframework.stereotype.Service
import org.springframework.beans.factory.annotation.Autowired

import  com.jos.dem.vetlog.model.User
import  com.jos.dem.vetlog.command.Command
import  com.jos.dem.vetlog.binder.UserBinder
import  com.jos.dem.vetlog.service.UserService
import  com.jos.dem.vetlog.repository.UserRepository

@Service
class UserServiceImpl implements UserService {

  @Autowired
  UserBinder userBinder
  @Autowired
  UserRepository userRepository

  User getUserByUsername(String username){
    userRepository.findByUsername(username)
  }

  void save(Command command){
    User user = userBinder.bindUser(command)
    userRepository.save(user)
  }

}
```

**UserRepository**

```groovy
package com.jos.dem.vetlog.repository

import com.jos.dem.vetlog.model.User
import org.springframework.data.jpa.repository.JpaRepository

interface UserRepository extends JpaRepository<User,Long> {

  User findByUsername(String username)

}
```

To download the project

```bash
git clone https://github.com/josdem/vetlog-spring-boot.git
git fetch
git checkout feature/7
```

To run the project, please read the [wiki](https://github.com/josdem/vetlog-spring-boot/wiki) and execute this command:

```bash
gradle bootRun -Dspring.config.location=$HOME/.vetlog-spring-boot/application-development.yml
```


[Return to the main article](/techtalk/spring)

