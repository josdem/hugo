+++
date = "2018-01-27T15:45:00-06:00"
tags = ["josdem", "techtalks","programming","technology","Spring Boot RESTful"]
categories = ["techtalk", "code"]
title = "Spring Boot RESTFul"
+++

The main idea behind creating a RESTfull application is that you will create logical resources (GET, POST, UPDATE and DELETE) consumed by clients usign HTTP request.

This resources should be nouns no verbs. In this example we will create a RESTful application using a person resource.

So we can have:

* GET /persons - Retrieves a list of persons
* GET /persons/1234 - Retrieves a specific person
* POST /persons - Creates a new person
* PUT /persons/1234 - Updates #1234 person
* DELETE /tickets/1234 - Deletes #1234 person


**NOTE:** If you need to know what tools you need to have installed in yout computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot)

Then execute this command in your terminal.

```bash
spring init --dependencies=web --language=groovy --build=gradle spring-boot-restful
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

Here is how looks our person model

```groovy
package com.jos.dem.springboot.restful.model

class Person {
  String nickname
  String email
}
```

Netx let's create a `PersonService` to delegate all operations needed

```groovy
package com.jos.dem.springboot.restful.service

import com.jos.dem.springboot.restful.model.Person

interface PersonService {
  List<Person> getPersons()
  Person getPerson(String uuid)
  Person create(Person person)
  Person update(Person person)
  Person delete(String uuid)
}
```

Then let's look our controller who receives client's request and provide client's response

```groovy
package com.jos.dem.springboot.restful.controller

import static org.springframework.web.bind.annotation.RequestMethod.GET
import static org.springframework.web.bind.annotation.RequestMethod.POST
import static org.springframework.web.bind.annotation.RequestMethod.PUT
import static org.springframework.web.bind.annotation.RequestMethod.DELETE

import org.springframework.web.bind.annotation.RestController
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.web.bind.annotation.ResponseBody
import org.springframework.web.bind.annotation.ResponseStatus
import org.springframework.web.bind.annotation.PathVariable
import org.springframework.web.bind.annotation.ExceptionHandler
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.http.ResponseEntity
import org.springframework.http.HttpStatus

import com.jos.dem.springboot.restful.service.PersonService
import com.jos.dem.springboot.restful.model.Person
import com.jos.dem.springboot.restful.exception.BusinessException

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@RestController
@RequestMapping('/persons/**')
class PersonController {

  Logger log = LoggerFactory.getLogger(this.class)

  @Autowired
  PersonService personService

  @RequestMapping(method=GET)
  @ResponseBody
  List<Person> getPersons(){
    log.info 'Listing all persons'
    personService.getPersons()
  }

  @RequestMapping(value='/{uuid}' ,method=GET)
  @ResponseBody
  Person getPerson(@PathVariable String uuid){
    log.info "Getting person by uuid: ${uuid}"
    personService.getPerson(uuid)
  }


  @RequestMapping(method=POST, consumes='application/json')
  @ResponseBody
  Person createPerson(@RequestBody Person person){
    log.info "Creating new person: ${person.nickname}"
    personService.create(person)
  }

  @RequestMapping(method=PUT, consumes='application/json')
  @ResponseBody
  Person updatePerson(@RequestBody Person person){
    log.info "Updating person: ${person.nickname}"
    personService.update(person)
  }

  @RequestMapping(value='/{uuid}', method=DELETE)
  @ResponseBody
  Person deletePerson(@PathVariable String uuid){
    log.info "Deleting person with uuid: ${uuid}"
    personService.delete(uuid)
  }

  @ResponseStatus(value=HttpStatus.UNAUTHORIZED)
  @ExceptionHandler(BusinessException.class)
  ResponseEntity<String> handleException(BusinessException be){
    return new ResponseEntity<String>(be.message, HttpStatus.UNAUTHORIZED)
  }

}
```

`ExceptionHandler` is a strategy to catch any exception that possibly occurs, receives an `BusinessException` who it is actually a wrapper exception. For more information regading this strategy, please refer my previous post: [Spring Boot AOP](/techtalk/spring_boot_aop)

To get list of persons:

```bash
curl  -v 'http://localhost:8080/persons/'
```

To get a person by uuid:

```bash
curl  -v 'http://localhost:8080/persons/1234'
```

To create a new person:

```bash
curl -H "Content-Type: application/json" -v -X POST -d '{"nickname":"josdem","email":"joseluis.delacruz@gmai.com"}' 'http://localhost:8080/persons/'
```

To update a person:

```bash
curl -H "Content-Type: application/json" -v -X PUT -d '{"nickname":"josdem","email":"joseluis.delacruz@gmai.com"}' 'http://localhost:8080/persons/'
```

To delete a person by uuid:

```bash
curl -v -X DELETE 'http://localhost:8080/persons/1234'
```

This is how get all persons response looks like:

```bash
Trying 127.0.0.1...
TCP_NODELAY set
Connected to localhost (127.0.0.1) port 8080 (#0)

> GET /persons/ HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.55.1
> Accept: */*

< HTTP/1.1 200
< Content-Type: application/json;charset=UTF-8
< Transfer-Encoding: chunked
< Date: Sat, 27 Jan 2018 20:28:31 GMT

Connection #0 to host localhost left intact
[{"nickname":"josdem","email":"joseluis.delacruz@gmail.com"}]
```

To Download the Project:

```bash
git clone https://github.com/josdem/spring-boot-restful.git
```

To Run the project:

```bash
gradle bootRun
```

[Return to the main article](/techtalk/spring)
