+++
title = "Spring Boot Flyway"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology","flyway","spring boot", "Gradle", "control version database"]
date = "2017-06-05T14:45:09-05:00"
description = "You know the benefits using version of control in software development such as Git or Subversion. This time I will show you Flyway to manage version control for your database, so you can track schema's evolution across all your environments with ease using Gradle and Spring Boot."
+++

You know the benefits using version of control in software development such as [Git](https://git-scm.com/) or [Subversion](https://subversion.apache.org/). This time I will show you [Flyway](https://flywaydb.org/) to manage version control for your database, so you can track schema's evolution across all your environments with ease using [Gradle](https://gradle.org/) and Spring Boot.

Let’s start creating a new Spring Boot project with web and jpa dependencies:

```bash
spring init --dependencies=web,jpa --language=groovy --build=gradle spring-boot-flyway
```

This is the `build.gradle` file generated:

```groovy
buildscript {
  ext {
    springBootVersion = '1.5.12.RELEASE'
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
  compile 'org.springframework.boot:spring-boot-starter-web'
  compile 'org.springframework.boot:spring-boot-starter-data-jpa'
  compile 'mysql:mysql-connector-java:5.1.34'
  compile 'org.codehaus.groovy:groovy'
  testCompile 'org.springframework.boot:spring-boot-starter-test'
}
```

Then let's add Flyway plugin to the `gradle.properties` to connect to MySQL database:

```groovy
plugins {
  id "org.flywaydb.flyway" version "5.0.7"
}

flyway {
  url = 'jdbc:mysql://localhost:3306/flyway_demo'
  user = 'flywayUser'
  password = 'flywaySecret'
}
```

Now let's create first migration called `src/main/resources/db/migration/V1__person_create.sql`:

```sql
DROP TABLE IF EXISTS `person`;
CREATE TABLE `person` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `firstname` varchar(255) NOT NULL,
  `lastname` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
);
```

Secondly, let's add a second migration called `src/main/resources/db/migration/V2__person_insert.sql`:

```sql
INSERT INTO `person` VALUES (1, 'Jose Luis', 'De la Cruz Morales'), (2, 'Eric', 'Haddad')
```

Let's run Flyway to migrate our database using gradle:

```bash
gradle flywayMigrate -i
```

If all went well, you should see the following output:

```bash
Flyway Community Edition 5.0.7 by Boxfuse
Database: jdbc:mysql://localhost:3306/flyway_demo (MySQL 5.7)
Successfully validated 2 migrations (execution time 00:00.039s)
Current version of schema `flyway_demo`: << Empty Schema >>
Migrating schema `flyway_demo` to version 1 - person create
Migrating schema `flyway_demo` to version 2 - person insert
Successfully applied 2 migrations to schema `flyway_demo` (execution time 00:00.985s)
```

We can do `flywayMigrate` task as dependent to `bootRun` task in gradle as follow:

```groovy
bootRun.dependsOn rootProject.tasks['flywayMigrate']
```


Flyway uses `VX__description.sql` convention names. We are using `application.properties` to specify embedded database driver class, credentials and ddl (Data Definition Language) strategy.

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/flyway_demo
spring.datasource.username=flywayUser
spring.datasource.password=secret
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

spring.jpa.generate-ddl=false
spring.jpa.hibernate.ddl-auto=none
```


JPA has features for DDL generation, and these can be set up to run on startup against the database. This is controlled through two last previous properties:

* `spring.jpa.generate-ddl` (boolean) switches the feature on and off and is vendor independent.
* `spring.jpa.hibernate.ddl-auto` (enum) is a Hibernate feature that controls the behavior in a more fine-grained way. This features are: `create`, `create-drop`, `validate`, and `update`.

In production or using Flyway version control it is highly recommended you use `none` or simply do not specify this property.

Here is our `Person` entity:

```groovy
package com.jos.dem.springboot.flyway.model

import static javax.persistence.GenerationType.AUTO

import javax.persistence.Id
import javax.persistence.Column
import javax.persistence.Entity
import javax.persistence.GeneratedValue

@Entity
class Person {

  @Id
  @GeneratedValue(strategy=AUTO)
  Long id

  @Column(nullable = false)
  String firstname

  @Column(nullable = false)
  String lastname

}
```

To browse the project go [here](https://github.com/josdem/spring-boot-flyway), to download the project:

```bash
git clone https://github.com/josdem/spring-boot-flyway.git
```

To run the project:

```bash
gradle bootRun
```


[Return to the main article](/techtalk/spring#Spring_Boot)
