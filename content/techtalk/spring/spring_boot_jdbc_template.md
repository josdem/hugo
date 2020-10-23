+++
title = "Spring Boot JDBC Template"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology", "spring boot", "spring boot jdbc template"]
date = "2017-06-30T11:03:38-05:00"
description = "This time I will show you how to use JDBC template in a Spring Boot application. Spring JdbcTemplate is a powerful mechanism to connect to a relational database and execute SQL queries."
+++

Spring JDBC template provide an abstraction that makes easy for you to implement relational database operations within a Spring Boot application. Spring [JdbcTemplate](https://docs.spring.io/spring-framework/docs/1.0.0/api/org/springframework/jdbc/core/JdbcTemplate.html) is the central class in the JDBC core package.

* [Query for Multiple Rows](#QueryMultipleRows)
* [Query for Object](#QueryForObject)
* [Query for Update](#QueryForUpdate)

Let’s start creating a new Spring Boot project with web and jdbc dependencies:

```bash
spring init --dependencies=web,jdbc --language=java --build=gradle spring-boot-jdbc-template
```

This is the `build.gradle` file generated:


```groovy
plugins {
	id 'org.springframework.boot' version '2.3.4.RELEASE'
	id 'io.spring.dependency-management' version '1.0.10.RELEASE'
	id 'java'
}

group = 'com.jos.dem.springboot.jdbc'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 13

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter'
	implementation 'org.springframework.boot:spring-boot-starter-jdbc'
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation('org.springframework.boot:spring-boot-starter-test') {
		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
	testImplementation 'io.projectreactor:reactor-test'
}
```

Do not forget to add MySQL dependency to the `build.gradle` file:

```groovy
compile 'mysql:mysql-connector-java:8.0.15'
```

Let's create a `person` model object that represent person table in a MySQL database.

```java
package com.jos.dem.springboot.jdbc.model;

import lombok.Data;

@Data
public class Person {

	private long id;
	private String nickname;
	private String email;
	private int ranking;
}
```

As a precondition to execute this project we will need a MySQL database created in our local computer named `spring_boot_jdbc_template` with a person table.

```sql
CREATE TABLE `person` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `nickname` varchar(255) NOT NULL,
  `email` varchar(255) NOT NULL,
  `ranking` int(11) NOT NULL,
  PRIMARY KEY (`id`)
);
```

And this data:

```sql
+----+----------+-----------------------------+---------+
| id | nickname | email                       | ranking |
+----+----------+-----------------------------+---------+
|  1 | josdem   | joseluis.delacruz@gmail.com |       5 |
|  2 | erich    | eric@email.com              |       5 |
|  3 | martinv  | martin@email.com            |       4 |
+----+----------+-----------------------------+---------+
```

<a name="QueryMultipleRows">
## Query for Multiple Rows
</a>


`JdbcTemplate` is the lowest level approach to manage databases other implementations uses it behind the scenes. From our end let's create a `PersonRepository` to use it.

```java
package com.jos.dem.springboot.jdbc.repository;

import com.jos.dem.springboot.jdbc.model.Person;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

import java.util.List;

@Slf4j
@Repository
@RequiredArgsConstructor
public class PersonRepository {

  private final JdbcTemplate jdbcTemplate;


  public List<Person> findAll() {
    return jdbcTemplate.query(
            "SELECT * FROM person",
            BeanPropertyRowMapper.newInstance(Person.class)
    );
  }

}
```

`BeanPropertyRowMapper` can maps a row’s column value to a property by matching their names in a model object. Now let's create a service to delegate data access to the repository:

```java
package com.jos.dem.springboot.jdbc.service;

import com.jos.dem.springboot.jdbc.model.Person;

import java.util.List;

public interface PersonService {

	List<Person> getPersons();

}
```

This is the implementation

```java
package com.jos.dem.springboot.jdbc.service.impl;

import com.jos.dem.springboot.jdbc.model.Person;
import com.jos.dem.springboot.jdbc.repository.PersonRepository;
import com.jos.dem.springboot.jdbc.service.PersonService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.util.List;

@Slf4j
@Service
@RequiredArgsConstructor
public class PersonServiceImpl implements PersonService {

  private final PersonRepository personRepository;

  public List<Person> getPersons() {
    log.info("Querying for getting persons");
    return personRepository.findAll();
  }

}
```

It's time to wire up our `PersonService` bean from the spring application context using `CommandLineRunner`:

```java
package com.jos.dem.springboot.jdbc;

import com.jos.dem.springboot.jdbc.model.Person;
import com.jos.dem.springboot.jdbc.service.PersonService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import java.util.List;

@Slf4j
@SpringBootApplication
public class JdbcApplication {

    public static void main(String[] args) {
        SpringApplication.run(JdbcApplication.class, args);
    }


    @Bean
    CommandLineRunner run(PersonService personService){
        return args -> {
            List<Person> persons = personService.getPersons();
            persons.forEach(person -> log.info("person: {}", person));
        };
    }

}
```

Don't forget to set datasource information in your `application.yml`

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost/spring_boot_jdbc_template?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
    username: username
    password: password
    driver-class-name: com.mysql.jdbc.Driver
```

<a name="QueryForObject">
## Query for Object
</a>

Here is an example how to query for a single domain object:

```java
public Person findByNickname(String nickname) {
  return jdbcTemplate.queryForObject(
    "SELECT * FROM person WHERE nickname = ?", new Object[]{nickname},
    BeanPropertyRowMapper.newInstance(Person.class)
  );
}
```

Notice that we are using an object array to set our parameter query.

<a name="QueryForUpdate">
## Query for Update
</a>

Update querying is very self explanatory.

```java
public int updateRanking(String nickname, int ranking) {
  return jdbcTemplate.update("UPDATE person SET ranking = ? WHERE nickname = ?", ranking, nickname);
}
```

Here is the complete `PersonRepository` example:

```java
package com.jos.dem.springboot.jdbc.repository;

import com.jos.dem.springboot.jdbc.model.Person;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

import java.util.List;

@Slf4j
@Repository
@RequiredArgsConstructor
public class PersonRepository {

  private final JdbcTemplate jdbcTemplate;


  public Person findByNickname(String nickname) {
    return jdbcTemplate.queryForObject(
            "SELECT * FROM person WHERE nickname = ?", new Object[]{nickname},
            BeanPropertyRowMapper.newInstance(Person.class)
    );
  }


  public List<Person> findAll() {
    return jdbcTemplate.query(
            "SELECT * FROM person",
            BeanPropertyRowMapper.newInstance(Person.class)
    );
  }

  public int updateRanking(String nickname, int ranking) {
    return jdbcTemplate.update("UPDATE person SET ranking = ? WHERE nickname = ?", ranking, nickname);
  }

}
```

To browse the project go [here](https://github.com/josdem/spring-boot-jdbc-template), to download the project

```bash
git clone https://github.com/josdem/spring-boot-jdbc-template.git
```

To run the project

```bash
gradle bootRun
```

[Return to the main article](/techtalk/spring#Spring_Boot)
