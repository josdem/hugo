+++
date = "2017-06-05T14:45:09-05:00"
title = "Spring Boot Flyway"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology","flyway","spring boot", "Gradle"]

+++


This time I will show you how to integrate [Flyway](https://flywaydb.org/) using [Gradle](https://gradle.org/) in Spring Boot. First you need to set Flyway plugin in your `gradle.properties` 

```groovy
buildscript {
	ext {
		springBootVersion = '1.5.3.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

plugins {
  id "org.flywaydb.flyway" version "4.2.0"
}

flyway {
  url = 'jdbc:mysql://localhost:3306/flyway_demo'
  user = 'flywayUser'
  password = 'secret'
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

bootRun.dependsOn rootProject.tasks['flywayMigrate']
```

The last line we added `flywayMigrate` as dependency to `bootRun` task. `flywayMigrate` migrates the schema to the latest version. Flyway will create the metadata table automatically if it doesn't exist. Flyway will read the schema form `${PROJECT_HOME}/src/main/resources/db/migration`

```sql
DROP TABLE IF EXISTS `person`;
CREATE TABLE `person` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `firstname` varchar(255) NOT NULL,
  `lastname` varchar(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

This is our `V1__master_changelog.sql` Flyway uses `VX__description.sql` convention names.

We are using `application.properties` to specify embedded database driver class.

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/flyway_demo
spring.datasource.username=flywayUser
spring.datasource.password=secret
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

spring.jpa.generate-ddl=true
spring.jpa.hibernate.ddl-auto=validate
```

Here is our Person entity

```groovy
package com.jos.dem.flyway.model

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

To Run the project execute in your command line:

```bash
gradle bootRun
```

this will create the Person table in the `flyway_demo` database. Finally add a new migration file named: `V1.2__insert_persons.sql`

```sql
INSERT INTO `person` VALUES (1, 'Jose Luis', 'De la Cruz Morales'), (2, 'Eric', 'Haddad')
```

Execute again the project and you should see the changelog applied.

To download the project:

```bash
git clone https://github.com/josdem/spring-boot-flyway.git
```

[Return to the main article](/techtalk/spring)