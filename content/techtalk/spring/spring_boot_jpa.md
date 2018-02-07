+++
title = "Spring Boot JPA"
categories = ["techtalk","code"]
tags = ["josdem","techtalks","programming","technology","spring boot","spring boot jpa", "groovy", "gradle"]
date = "2018-02-07T12:33:50-06:00"

+++

The key goal of Spring Data repository abstraction is to significantly reduce the amount of code required to implement data access layers for various persistence stores.

In this example I will show you how to create a JPA repository to save and retrieve all `Person` model.


**NOTE:** If you need to know what tools you need to have installed in yout computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot)

Then execute this command in your terminal.

```bash
spring init --dependencies=web --language=groovy --build=gradle spring-boot-jpa
```

This is the `build.gradle file` generated:

```groovy
buildscript {
	ext {
		springBootVersion = '1.5.9.RELEASE'
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

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}

dependencies {
	compile('org.springframework.boot:spring-boot-starter-web')
	compile('org.codehaus.groovy:groovy')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}
```


This is our `Person` model

```groovy
package com.jos.dem.springboot.jpa.model

import static javax.persistence.GenerationType.AUTO

import javax.persistence.Id
import javax.persistence.Entity
import javax.persistence.Column
import javax.persistence.GeneratedValue

@Entity
class Person{

	@Id
	@GeneratedValue(strategy=AUTO)
  Long id

  @Column(nullable=false)
  String nickname
  
  @Column(unique=true, nullable=false)
  String email
	
}
```

And here is our `PersonRepository`

```groovy
package com.jos.dem.springboot.jpa.repository

import org.springframework.data.jpa.repository.JpaRepository
import com.jos.dem.springboot.jpa.model.Person

interface PersonRepository extends JpaRepository<Person, Long>{

	Person save(Person person)
	List<Person> findAll()
	
}
```

To Download the Project:

```bash
git clone https://github.com/josdem/spring-boot-jpa.git
git fetch
git checkout feature/jpa
```

To Run the project:

```bash
gradle bootRun
```

[Return to the main article](/techtalk/spring)