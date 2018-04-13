+++
title =  "Spring Boot Geb"
date = "2018-04-12T21:16:59-05:00"
tags = ["josdem", "techtalks","programming","technology","Geb Framework", "Geb and Spock", "BDD", "Functional Testing"]
categories = ["techtalk", "code"]
+++

BDD (Behavior-driven development) is a technique very similar to implement UAT (User Acceptance Testing) in a software project. Usually is a good idea to use BDD to reprecent how users can define application behaviour, so in that way you can represent user stories in test scenarios aka. feature testing.

This time I am going to show you how integrate [Geb](http://www.gebish.org/) to a Spring Boot application, Geb is a very powerful testing framework written in the Groovy programming language, which follows the BDD methodology.

Let's start creating a new Spring Boot project with Webflux, Lombok and Thymeleaf as dependencies:

```bash
spring init --dependencies=webflux,lombok,thymeleaf --build=gradle --language=java spring-boot-geb
```

Here is the complete `build.gradle` file generated:

```groovy
buildscript {
  ext {
    springBootVersion = '2.0.1.RELEASE'
  }
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  }
}

apply plugin: 'java'
apply plugin: 'groovy'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.geb'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
  mavenCentral()
}

dependencies {
  compile('org.springframework.boot:spring-boot-starter-webflux')
  compile('org.springframework.boot:spring-boot-starter-thymeleaf')
  compileOnly('org.projectlombok:lombok')
  testCompile('org.springframework.boot:spring-boot-starter-test')
  testCompile('io.projectreactor:reactor-test')
}
```

Now let's create a simple POJO retrieve information from an end point using Spring WebFlux.

```java
package com.jos.dem.springboot.geb.model;

import lombok.AllArgsConstructor;
import lombok.NoArgsConstructor;
import lombok.ToString;
import lombok.Data;

@AllArgsConstructor
@ToString
@NoArgsConstructor
@Data
public class Person {

  private String nickname;
  private String email;

}
```

Lombok is a great tool to avoid boilerplate code, for knowing more please go [here](https://projectlombok.org/)

Next step is to create methods to list, create and save persons:

```java
package com.jos.dem.springboot.geb.controller;

import static org.springframework.web.bind.annotation.RequestMethod.GET;
import static org.springframework.web.bind.annotation.RequestMethod.POST;

import com.jos.dem.springboot.geb.model.Person;
import com.jos.dem.springboot.geb.service.PersonService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class PersonController {

  @Autowired
  private PersonService personService;

  Logger log = LoggerFactory.getLogger(this.getClass());

  @RequestMapping(method=GET)
  public String persons(final Model model){
    log.info("Listing persons");
    model.addAttribute("persons", personService.getAll());
    return "list";
  }

  @RequestMapping(method=GET ,value="create")
  public String create(final Model model){
    log.info("Creating person");
    model.addAttribute("person", new Person());
    return "create";
  }

  @RequestMapping(method=POST)
  public String save(final Person person, final Model model){
    log.info("Saving person");
    personService.save(person);
    model.addAttribute("persons", personService.getAll());
    return "list";
  }

}
```

In order to complete our basement let’s create PersonService to bring data:

```java
package com.jos.dem.springboot.geb.service;

import com.jos.dem.springboot.geb.model.Person;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

public interface PersonService {
  Flux<Person> getAll();
  Mono<Person> getByNickname(String nickname);
  void save(Person person);
}
```

Implementation:

```java
package com.jos.dem.springboot.geb.service.impl;

import java.util.HashMap;
import java.util.Map;

import com.jos.dem.springboot.geb.model.Person;
import com.jos.dem.springboot.geb.service.PersonService;

import org.springframework.stereotype.Service;

import reactor.core.publisher.Flux;
import reactor.core.publisher.Mono;

@Service
public class PersonServiceImpl implements PersonService {

  private Map<String, Person> persons = new HashMap<String, Person>();

  public Flux<Person> getAll(){
    return Flux.fromIterable(persons.values());    
  }

  public Mono<Person> getByNickname(String nickname){
    return Mono.just(persons.get(nickname));
  }

  public void save(Person person){
    persons.put(person.getNickname(), person);
  }

}
```

Now all is set in our service, let’s continue adding Groovy, Geb and Spock as dependencies in our `build.gradle` file:

```groovy
compile('org.codehaus.groovy:groovy-all')
testCompile('org.gebish:geb-spock:2.1')
testCompile('org.seleniumhq.selenium:selenium-firefox-driver:3.11.0')
testCompile('org.spockframework:spock-spring')
testCompile('org.spockframework:spock-spring:1.1-groovy-2.4')
testCompile("io.github.bonigarcia:webdrivermanager:1.5.0") {
  exclude group: 'org.seleniumhq.selenium'
}
```

Geb by convention will look for a `src/test/resources/GebConfig.groovy` file which contains web driver definition Firefox by default and reports directory where Geb saves the screenshots and HTML dumps at the end of each test:

```groovy
import io.github.bonigarcia.wdm.FirefoxDriverManager
import org.openqa.selenium.firefox.FirefoxDriver

reportsDir = 'build/test-reports'

atCheckWaiting = true

driver = {
  FirefoxDriverManager.getInstance().setup()
  new FirefoxDriver()
}
```

Next step is to define page object, since is a good idea to separate page layout from logic behaviour:

```groovy
package com.jos.dem.springboot.geb.pages

import geb.Page

class PersonList extends Page {

  static url = "list"
  static at = { title == "Person List" }
  static content = {
  }

}
```

Now we can define our test behaviour using [Spock Framework](http://spockframework.org/)

```groovy
package com.jos.dem.springboot.geb

import geb.spock.GebReportingSpec
import com.jos.dem.springboot.geb.pages.PersonList

class CreatePersonSpec extends GebReportingSpec {

  void 'should create person'() {
    given:
      go "http://localhost:8080/create"

    when:
      $("input", name: "nickname").value("josdem")
      $("input", name: "email").value("joseluis.delacruz@gmail.com")
      $("button", name: "submit").click()

    then:
      at PersonList
    }

}
```

This test represent happy path:

* **given:** A url and go instruction that directs the web driver to the page.
* **when:** Fills out name and email text fields in the form and clicks the submit button. 
* **then:** Is just making sure we going to the right place. We could add any other assertions.

Geb is using a JQuery like style to access to our html elements, Geb call it [Navigator API](http://www.gebish.org/manual/current/#navigator).

To download the project:

```bash
git clone https://github.com/josdem/spring_boot_geb.git
```

To run the project:

```bash
gradle bootRun
```

To test the project:

```bash
gradle test
```

[Return to the main article](/techtalk/spring)
