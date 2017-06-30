+++
categories = ["techtalk","code"]
tags = ["josdem","techtalks","programming","technology", "spring boot", "spring boot jdbc template"]
date = "2017-06-30T11:03:38-05:00"
title = "Spring Boot JDBC Template"

+++

This time I will show you how to use JDBC template in a Spring Boot application, first you need to integrate JDBC template in your `build.gradle` file.


```groovy
buildscript {
  ext {
    springBootVersion = '1.5.4.RELEASE'
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

version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
  mavenCentral()
}


dependencies {
  compile 'org.springframework.boot:spring-boot-starter'
  compile 'org.springframework.boot:spring-boot-starter-jdbc'
  compile 'org.springframework:spring-jdbc:4.3.9.RELEASE'
  compile 'org.codehaus.groovy:groovy'
  compile 'mysql:mysql-connector-java:5.1.34'
  testCompile 'org.springframework.boot:spring-boot-starter-test'
}
```

Then we will create a `person` model object

```groovy
package com.jos.dem.jdbc.model

class Person {

  Long id	
  String nickname
  String email
  Integer ranking
	
}
```

Let's suppose you have a MySQL database called `spring_boot_jdbc_template` with a person table.

```sql
CREATE TABLE `person` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `nickname` varchar(255) NOT NULL,
  `email` varchar(255) NOT NULL,
  `ranking` int(11) NOT NULL,
  PRIMARY KEY (`id`)
)
```

With this data:

```sql
+----+----------+-----------------------------+---------+
| id | nickname | email                       | ranking |
+----+----------+-----------------------------+---------+
|  1 | josdem   | joseluis.delacruz@gmail.com |       5 |
|  2 | erich    | eric@email.com              |       5 |
|  3 | martinv  | martin@email.com            |       4 |
+----+----------+-----------------------------+---------+
```

Spring provides a template class called `JdbcTemplate` that makes it easy to work with SQL relational databases and JDBC, so let's create a `PersonRepository`

```groovy
package com.jos.dem.jdbc.repository

import org.springframework.stereotype.Repository
import org.springframework.jdbc.core.JdbcTemplate
import org.springframework.jdbc.core.BeanPropertyRowMapper
import org.springframework.beans.factory.annotation.Autowired

import com.jos.dem.jdbc.model.Person

@Repository
class PersonRepository {

  @Autowired
  JdbcTemplate jdbcTemplate

  List<Person> findAll() {
    jdbcTemplate.query(
      'SELECT * FROM person',
      BeanPropertyRowMapper.newInstance(Person.class)
    )
  }

}
```

`BeanPropertyRowMapper` can maps a row’s column value to a property by matching their names in a model object.

Now let's create a service to delegate data access to the repository:

```groovy
package com.jos.dem.jdbc.service

import com.jos.dem.jdbc.model.Person

interface PersonService {

  List<Person> getPersons()

}
```

This is the implementation

```groovy
package com.jos.dem.jdbc.service.impl

import org.springframework.stereotype.Service
import org.springframework.beans.factory.annotation.Autowired

import com.jos.dem.jdbc.model.Person
import com.jos.dem.jdbc.service.PersonService
import com.jos.dem.jdbc.repository.PersonRepository

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@Service
class PersonServiceImpl implements PersonService {

  @Autowired
  PersonRepository personRepository

  Logger log = LoggerFactory.getLogger(this.class)

  List<Person> getPersons() {
    log.info 'Querying for getting persons'
    personRepository.findAll()
  }

}
```

Finally we get `PersonService` bean from the spring application context and execute the `getPersons()` method:

```groovy
package com.jos.dem.jdbc

import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.context.ConfigurableApplicationContext

import com.jos.dem.jdbc.model.Person
import com.jos.dem.jdbc.service.PersonService

@SpringBootApplication
class JdbcApplication {

  static void main(String[] args) {
	ConfigurableApplicationContext context = SpringApplication.run JdbcApplication, args
    List<Person> persons = context.getBean(PersonService.class).getPersons()
    persons.each {
      println "person: ${it.dump()}"
    }
  }

}
```

Don't forget to set datasource information in your `application.properties`

```properties
spring.datasource.url=jdbc:mysql://localhost/spring_boot_jdbc_template
spring.datasource.username=username
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
``` 

To download the project

```bash
git clone https://github.com/josdem/spring-boot-jdbc-template.git
```

To run the project

```bash
gradle bootRun
```

[Return to the main article](/techtalk/spring)