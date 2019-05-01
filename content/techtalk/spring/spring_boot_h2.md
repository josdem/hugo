+++
title =  "Spring Boot H2"
description = "Spring_boot_h2"
date = "2019-05-01T19:31:24-04:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

In this technical post we will review how to integrate a [H2](https://www.h2database.com/html/main.html) un memory data base in a Spring Webflux application. Please read this previous [Spring Webflux Basics](/techtalk/spring/spring_webflux_basics) before conitnue with this information. Then, let’s create a new Spring Boot project with Webflux, Lombok, JPA and H2 as dependencies:

```bash
spring init --dependencies=webflux,lombok,jpa,h2 --language=java --build=gradle spring-boot-h2
```

Here is the complete `build.gradle` file generated:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.4.RELEASE'
  id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.h2'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
  mavenCentral()
}

dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
  implementation 'org.springframework.boot:spring-boot-starter-webflux'
  runtimeOnly 'com.h2database:h2'
  compileOnly('org.projectlombok:lombok')
  annotationProcessor('org.projectlombok:lombok')
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
  testImplementation 'io.projectreactor:reactor-test'
}
```

Let's start by creating a person model so that we can store this entity in our H2 database

```java
package com.jos.dem.springboot.h2.model;

import javax.persistence.Id;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;

import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.Getter;
import lombok.Setter;

@Entity
@NoArgsConstructor
@AllArgsConstructor
@Data
public class Person {

	@Id
	@GeneratedValue
	private Long id;
	private String nickname;
	private String email;

}
```

To run the project:

```bash
gradle bootRun
```

To browse the project go [here](https://github.com/josdem/spring-boot-h2), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-h2.git
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)

