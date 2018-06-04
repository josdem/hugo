+++
title =  "Spring Boot Ehcache"
description = "Ehcache is an open source, standards-based cache that boosts performance and It's the most widely-used Java-based cache because it's robust, proven, full-featured, and integrates with other popular libraries and frameworks."
date = "2018-06-03T18:21:44-05:00"
tags = ["josdem", "techtalks","programming","technology", "ehcache"]
categories = ["techtalk", "code", "spring boot"]
+++

[Ehcache](http://www.ehcache.org/) is an open source, standards-based cache that boosts performance. It's the most widely-used Java-based cache because it's robust, proven, full-featured, and integrates with other popular libraries and frameworks. In this example I will show you how to use Ehcache in a Spring Boot application. **NOTE:** If you need to know what tools you need to have installed in your computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot)

Then execute this command in your terminal.

```bash
spring init --dependencies=web,data-jpa,lombok --language=groovy --build=gradle spring-boot-ehcache
```

This is the `build.gradle file` generated:

```groovy
buildscript {
	ext {
		springBootVersion = '2.0.2.RELEASE'
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

group = 'com.jos.dem.springboot.cache'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencies {
  compile 'org.springframework.boot:spring-boot-starter-web'
  compile 'org.springframework.boot:spring-boot-starter-data-jpa'
  compile 'org.projectlombok:lombok'
  testCompile 'org.springframework.boot:spring-boot-starter-test'
}
```

Then add Ehcache and MySQL dependencies to you `build.gradle` file.

```groovy
compile 'net.sf.ehcache:ehcache-core:2.6.11'
compile 'mysql:mysql-connector-java:5.1.34'
```

Now let's create an entity to store and retrieve information from our database.

```java
package com.jos.dem.springboot.cache.model;

import javax.persistence.Id;
import javax.persistence.Entity;

import lombok.AllArgsConstructor;
import lombok.NoArgsConstructor;
import lombok.ToString;
import lombok.Data;

@Data
@ToString
@AllArgsConstructor
@NoArgsConstructor
@Entity
public class Person {

  @Id
  private Long id;
  private String nickname;
  private String email;

}
```

Lombok is a great tool to avoid boilerplate code, for knowing more please go [here](https://projectlombok.org/). Since we are using Spring Data, let's create a JPA Person repository.

```java
package com.jos.dem.springboot.cache.repository;

import java.util.List;

import org.springframework.data.jpa.repository.JpaRepository;
import com.jos.dem.springboot.cache.model.Person;

public interface PersonRepository extends JpaRepository<Person, Long>{

  Person save(Person person);
  List<Person> findAll();
  void deleteAll();

}
```

Next, we are going to use `CommandLineRunner` to save some person entities. The `CommandLineRunner` is a call back interface in Spring Boot, when Spring Boot starts will call it and pass in args through a `run()` internal method.

```java
package com.jos.dem.springboot.cache;

import java.util.List;
import java.util.Arrays;

import org.springframework.context.annotation.Bean;
import org.springframework.boot.CommandLineRunner;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import com.jos.dem.springboot.cache.model.Person;
import com.jos.dem.springboot.cache.repository.PersonRepository;

@SpringBootApplication
@EnableCaching
public class CacheApplication {

  private List<Person> persons = Arrays.asList(
    new Person(1L, "josdem", "joseluis.delacruz@gmail.com"),
    new Person(2L, "tgrip", "tgrip@email.com"),
    new Person(3L, "edzero", "edzero@email.com"),
    new Person(4L, "jeduan", "jeduan@email.com"),
    new Person(5L, "siedrix", "siedrix@email.com")
  );


  public static void main(String[] args) {
    SpringApplication.run(CacheApplication.class, args);
  }

  @Bean
  CommandLineRunner start(PersonRepository personRepository){
    return args -> {
      personRepository.deleteAll();
      persons.forEach(person -> personRepository.save(person));
    };
  }

}
```

In our spring boot application class we are enabling caching itself with the `@EnableCaching` annotation. Setting up caching for a controller method is quite easy. Let's create a `PersonController` and some method to retrieve all persons, so we can configure cache on it:

```java
package com.jos.dem.springboot.cache.controller;

import java.util.Date;
import java.util.List;

import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.Cacheable;
import com.jos.dem.springboot.cache.model.Person;
import com.jos.dem.springboot.cache.service.PersonService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@RestController
public class PersonController {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Autowired
  private PersonService personService;

  @RequestMapping("/")
  @Cacheable("persons")
  public List<Person> index(){
    log.info("Getting all persons");
    long start = new Date().getTime();
    List<Person> persons = personService.getAll();
    log.info("Finishing getting all persons at: " + (new Date().getTime() - start) + " milliseconds");
    return persons;
  }

}
```

Ehcache has a lot configuration options, you can configure the cache size, time to live, if it should overflow to disk, etc. In this example I will just create a simple in memory cache. Create a file called `ehcache.xml` in the `src/main/resources` folder:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="http://www.ehcache.org/ehcache.xsd"
  updateCheck="true" monitoring="autodetect" dynamicConfig="true">

  <cache name="persons"
    maxElementsInMemory="1000" eternal="false"
    overflowToDisk="false"
    timeToLiveSeconds="300" timeToIdleSeconds="0"
    memoryStoreEvictionPolicy="LFU" transactionalMode="off">
  </cache>

</ehcache>
```

As you can see, we created a `<cache>` with the name "persons", with 1000 items that can be stored in memory and a time to live in 5 minutes. The next step is to configure Spring Boot to use this configuration file along with the MySQL configuration, please add the following structure in the `application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost/spring_boot_ehcache
spring.datasource.username=username
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

spring.jpa.generate-ddl=true

spring.cache.ehcache.config=classpath:ehcache.xml
```

That's it, now if you start up your Spring Boot application and hit the index page at: [http://localhost:8080/](http://localhost:8080/) you will be able to see our cache in action.

```json
[
  {
    "id": 5,
    "nickname": "edrix",
    "email": "siedrix@email.com"
  },
  {
    "id": 4,
    "nickname": "jeduan",
    "email": "jeduan@email.com"
  },
  {
    "id": 3,
    "nickname": "edzero",
    "email": "edzero@email.com"
  },
  {
    "id": 2,
    "nickname": "tgrip",
    "email": "tgrip@email.com"
  },
  {
    "id": 1,
    "nickname": "josdem",
    "email": "joseluis.delacruz@gmail.com"
  }
]
```

To download the project:

```bash
git clone https://github.com/josdem/spring-boot-ehcache.git
```

To run the project:

```bash
gradle bootRun
```

[Return to the main article](/techtalk/spring)
