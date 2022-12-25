+++
title = "Spring Boot Flyway"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology","flyway","spring boot", "Gradle", "control version database"]
date = "2017-06-05T14:45:09-05:00"
description = "You know the benefits using version of control in software development such as Git or Subversion. This time I will show you Flyway to manage version control for your database, so you can track schema's evolution across all your environments with ease using Gradle and Spring Boot."
+++

In the same way we find benefits using version of control in software development with [Git](https://git-scm.com/), we can versioning our database so that we can manage changes in both schema and information. Let me introduce [Flyway](https://flywaydb.org/) an open source project that help us to do database migrations easily, that's it, how cool would be to see our database evolution across all our development life cycle?. In this example we wil be using [Gradle](https://gradle.org/) and Spring Boot. Let’s start creating a new Spring Boot project with web and jpa dependencies:

```bash
spring init --dependencies=web,mysql,jpa --language=java --build=gradle --type=gradle-project spring-boot-flyway
```

This is the `build.gradle` file generated:

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '2.7.7'
    id 'io.spring.dependency-management' version '1.1.0'
}

group = 'com.jos.dem.springboot.flyway'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '17'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    runtimeOnly 'mysql:mysql-connector-java'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

Then let's add Flyway plugin to the `build.gradle` file to connect to MySQL database:

```groovy
buildscript {
    dependencies {
        classpath 'org.flywaydb:flyway-mysql:9.8.1'
    }
}

plugins {
    id 'java'
    id 'org.springframework.boot' version '2.7.7'
    id "org.flywaydb.flyway" version "9.8.1"
    id 'io.spring.dependency-management' version '1.1.0'
}

flyway {
  url = 'jdbc:mysql://localhost:3306/flyway_demo'
  user = 'databaseUser'
  password = 'databasePassword'
}
```

Now let's create first migration called `src/main/resources/db/migration/V1__person_create.sql`:

```sql
DROP TABLE IF EXISTS `person`;
CREATE TABLE `person` (
  `id` int NOT NULL AUTO_INCREMENT,
  `first_name` varchar(255) NOT NULL,
  `last_name` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
);
```

Secondly, let's add a second migration called `src/main/resources/db/migration/V2__person_insert.sql`:

```sql
INSERT INTO `person` VALUES (1, 'Jose', 'Morales'), (2, 'Eric', 'Haddad')
```

Let's run Flyway to migrate our database using gradle:

```bash
gradle flywayMigrate -i
```

If all went well, you should see the following output:

```bash
Database: jdbc:mysql://localhost:3306/flyway_demo (MySQL 8.0)
Successfully validated 2 migrations (execution time 00:00.006s)
Creating Schema History table `flyway_demo`.`flyway_schema_history` ...
Current version of schema `flyway_demo`: << Empty Schema >>
Migrating schema `flyway_demo` to version "1 - person create"
DB: Unknown table 'flyway_demo.person' (SQL State: 42S02 - Error Code: 1051)
Migrating schema `flyway_demo` to version "2 - person insert"
Successfully applied 2 migrations to schema `flyway_demo`, now at version v2 (execution time 00:00.085s)
:flywayMigrate (Thread[Execution worker Thread 2,5,main]) completed. Took 0.865 secs.

BUILD SUCCESSFUL
```

We can do `flywayMigrate` task as dependent to `bootRun` task in gradle as follow:

```groovy
bootRun.dependsOn rootProject.tasks['flywayMigrate']
```

Here is our `Person` entity:

```groovy
package com.jos.dem.springboot.flyway.model;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

import static javax.persistence.GenerationType.AUTO;

@Entity
public class Person {

  @Id
  @GeneratedValue(strategy = AUTO)
  Long id;

  @Column(nullable = false)
  String firstName;

  @Column(nullable = false)
  String lastName;
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
