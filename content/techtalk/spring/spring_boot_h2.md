+++
title =  "Spring Boot H2"
description = "Spring_boot_h2"
date = "2019-05-01T19:31:24-04:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

In this technical post we will review how to integrate an [H2](https://www.h2database.com/html/main.html) in memory database in a Spring Webflux application. Please read this previous [Spring Webflux Basics](/techtalk/spring/spring_webflux_basics) before conitnue with this information. Then, let’s create a new Spring Boot project with Webflux, Lombok, JPA and H2 as dependencies:

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

And here is our `PersonRepository`, by extending `JpaRepository` we get a bunch of generic `CRUD` methods into our type that allows save, find all persons and so on, also will allow Spring Data JPA repository infrastructure to scan the classpath for this interface and create a Spring bean for it.

```java
package com.jos.dem.springboot.h2.repository;

import java.util.List;

import org.springframework.data.jpa.repository.JpaRepository;
import com.jos.dem.springboot.h2.model.Person;

public interface PersonRepository extends JpaRepository<Person, Long>{

  Person save(Person person);
  List<Person> findAll();

}
```

Next, we are going to use `CommandLineRunner` to populate our `person` table. The `CommandLineRunner` is a call back interface in Spring Boot, when Spring Boot starts will call it and pass in args through a `run()` internal method.

```java
package com.jos.dem.springboot.h2;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.domain.EntityScan;
import org.springframework.context.annotation.Bean;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

import com.jos.dem.springboot.h2.model.Person;
import com.jos.dem.springboot.h2.repository.PersonRepository;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@SpringBootApplication
@EntityScan("com.jos.dem.springboot.h2.model")
@EnableJpaRepositories("com.jos.dem.springboot.h2.repository")
public class H2Application {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  public static void main(String[] args) {
    SpringApplication.run(H2Application.class, args);
  }

  @Bean
  CommandLineRunner start(PersonRepository personRepository){
    return args -> {
      Person person  = new Person(1L, "josdem", "joseluis.delacruz@gmail.com");
      personRepository.save(person);
      personRepository.findAll().forEach(p -> log.info("person: {}", p));
    };
  }

}
```

H2 provides a web interface called H2 Console. Let’s enable that console in our `application.properties`.

```properties
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
spring.datasource.url=jdbc:h2:mem:persondb
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.platform=h2
```

The final step is to add `spring-boot-starter-web` dependency in order to get this H2 console work.

```groovy
implementation 'org.springframework.boot:spring-boot-starter-web'
```

Now, if you start our application, you should see this output

```bash
2019-05-02 09:14:42.378  INFO 8960 --- [main] com.jos.dem.springboot.h2.H2Application  : Started H2Application in 4.944 seconds (JVM running for 5.357)
2019-05-02 09:14:42.492  INFO 8960 --- [main] o.h.h.i.QueryTranslatorFactoryInitiator  : HHH000397: Using ASTQueryTranslatorFactory
2019-05-02 09:14:42.609  INFO 8960 --- [main] ication$$EnhancerBySpringCGLIB$$de299e81 : person: Person(id=1, nickname=josdem, email=joseluis.delacruz@gmail.com)
```

And the H2 console in this URL: [http://localhost:8080/h2-console](http://localhost:8080/h2-console)

To run the project:

```bash
gradle bootRun
```

To browse the project go [here](https://github.com/josdem/spring-boot-h2), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-h2.git
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)

