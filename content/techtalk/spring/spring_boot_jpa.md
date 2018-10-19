+++
title = "Spring Boot JPA"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology","spring boot","spring boot jpa", "groovy", "gradle"]
date = "2018-02-07T12:33:50-06:00"
description = "The key goal of Spring Data repository abstraction is to significantly reduce the amount of code required to implement data access layers for various persistence stores."
+++

The key goal of Spring Data repository abstraction is to significantly reduce the amount of code required to implement data access layers for various persistence stores. In this example I will show you how to create a JPA repository to save and retrieve all `Person` model.


**NOTE:** If you need to know what tools you need to have installed in yout computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring/spring_boot)

Then execute this command in your terminal.

```bash
spring init --dependencies=web,jpa,thymeleaf --language=java --build=gradle spring-boot-jpa
```

This is the `build.gradle file` generated:

```java
buildscript {
	ext {
		springBootVersion = '2.0.6.RELEASE'
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

group = 'com.jos.dem.springboot.jpa'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}


dependencies {
	compile('org.springframework.boot:spring-boot-starter-web')
	compile('org.springframework.boot:spring-boot-starter-thymeleaf')
	compile('org.springframework.boot:spring-boot-starter-data-jpa')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

Now add lombok and mysql-connector dependencies to your `build.gradle` file:

```groovy
compile("mysql:mysql-connector-java:5.1.34")
compileOnly('org.projectlombok:lombok')
```

Lombok is a great tool to avoid boilerplate code, for knowing more please go [here](https://projectlombok.org/). In this example, you store `Person` objects, annotated as a JPA entity.

```java
package com.jos.dem.springboot.jpa.model;

import static javax.persistence.GenerationType.AUTO;

import javax.persistence.Id;
import javax.persistence.Entity;
import javax.persistence.Column;
import javax.persistence.GeneratedValue;

import lombok.Data;

@Data
@Entity
public class Person {

  @Id
  @GeneratedValue(strategy=AUTO)
  private Long id;

  @Column(nullable=false)
  private String nickname;

  @Column(unique=true, nullable=false)
  private String email;

}
```

And here is our `PersonRepository`, by extending `JpaRepository` we get a bunch of generic `CRUD` methods into our type that allows save, find all persons and so on. Even we can define specific queries such as `findByNickname('josdem')`. Second, this will allow the Spring Data JPA repository infrastructure to scan the classpath for this interface and create a Spring bean for it.

```java
package com.jos.dem.springboot.jpa.repository;

import java.util.List;

import org.springframework.data.jpa.repository.JpaRepository;
import com.jos.dem.springboot.jpa.model.Person;

public interface PersonRepository extends JpaRepository<Person, Long>{

	Person save(Person person);
	List<Person> findAll();

}
```

Now, let's take a look to our controller.

```java
package com.jos.dem.springboot.jpa.controller;

import static org.springframework.web.bind.annotation.RequestMethod.GET;
import static org.springframework.web.bind.annotation.RequestMethod.POST;

import java.util.List;
import javax.validation.Valid;

import org.springframework.stereotype.Controller;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.validation.BindingResult;

import com.jos.dem.springboot.jpa.model.Person;
import com.jos.dem.springboot.jpa.command.Command;
import com.jos.dem.springboot.jpa.command.PersonCommand;
import com.jos.dem.springboot.jpa.repository.PersonRepository;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Controller
@RequestMapping("persons/**")
public class PersonController {

  @Autowired
  private PersonRepository personRepository;

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @RequestMapping(method=GET)
  public ModelAndView getAll(){
    log.info("Listing all persons");
    ModelAndView modelAndView = new ModelAndView("persons/list");
    List<Person> persons = personRepository.findAll();
    modelAndView.addObject("persons", persons);
    return modelAndView;
  }

  @RequestMapping(value="create", method=GET)
  public ModelAndView create(){
    log.info("Creating person");
    ModelAndView modelAndView = new ModelAndView("persons/create");
    Command personCommand = new PersonCommand();
    modelAndView.addObject("personCommand", personCommand);
    return modelAndView;
  }

  @RequestMapping(method=POST)
  public ModelAndView save(@Valid PersonCommand personCommand, BindingResult bindingResult){
    log.info("Registering new Person: " + personCommand.getNickname());
    ModelAndView modelAndView = new ModelAndView("persons/list");
    if(bindingResult.hasErrors()){
      modelAndView.setViewName("persons/create");
      modelAndView.addObject("personCommand", personCommand);
      return modelAndView;
    }
    Person person = new Person();
    person.setNickname(personCommand.getNickname());
    person.setEmail(personCommand.getEmail());
    personRepository.save(person);
    List<Person> persons = personRepository.findAll();
    modelAndView.addObject("persons", persons);
    return modelAndView;
	}

}
```

It is important to notice that we are using a `PersonCommand` to receive the person's requested information from a client.

```java
package com.jos.dem.springboot.jpa.command;

import javax.validation.constraints.Size;
import javax.validation.constraints.NotNull;
import org.hibernate.validator.constraints.Email;

import lombok.Data;

@Data
public class PersonCommand implements Command {

	@NotNull
	@Size(min=3, max=50)
	private String nickname;

	@Email
	@NotNull
	@Size(min=1, max=250)
	private String email;

}
```

That's it, we are using `javax.validation` and `org.hibernate.validator` to validate that the fields meets required constraints.

```java
package com.jos.dem.springboot.jpa.command;

import java.io.Serializable;

public interface Command extends Serializable{}
```

`Command` is just an interface that extends `java.io.Serializable`

```html
<html>
<body>
  <form id="create" th:action="@{/persons}" th:object="${personCommand}" method="post">
  	<label for="nickname">Nickname:</label>
  	<input type="text" name="nickname" th:field="*{nickname}" placeholder="nickname" id="nickname"/>
  	<label th:if="${#fields.hasErrors('nickname')}" th:errors="*{nickname}"></label>
  	<br/>
  	<label for="email">Email:</label>
  	<input type="text" name="email" th:field="*{email}" placeholder="email" id="email"/>
  	<label th:if="${#fields.hasErrors('email')}" th:errors="*{email}"></label>
  	<br/><br/>
  	<button id="btn-success" type="submit">Submit</button>
  </form>
</body>
</html>
```

So if any error is detected in user's information, we catch them using `bindingResult.hasErrors()` method at the controller and so we are able to show them using `<label th:if="${#fields.hasErrors('nickname')}" th:errors="*{nickname}"></label>` label in a html.

Here is out `application.properties` who is extenalized `MySQL` database credentials.

```properties
spring.datasource.url=jdbc:mysql://localhost/spring_boot_jpa
spring.datasource.username=username
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

spring.jpa.generate-ddl=true
```

Finally you can go to this address: [http://localhost:8080/persons/create](http://localhost:8080/persons/create) and you will see our create person form.

To browse the project go [here](https://github.com/josdem/spring-boot-jpa), to download the project:

```bash
git clone https://github.com/josdem/spring-boot-jpa.git
```

To Run the project:

```bash
gradle bootRun
```

[Return to the main article](/techtalk/spring#Spring_Boot)
