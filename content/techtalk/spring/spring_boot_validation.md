+++
categories = ["techtalk","code"]
tags = ["josdem","techtalks","programming","technology","java validation"]
date = "2017-01-08T09:54:20-06:00"
title = "Spring Boot Validation"

+++

Spring has a `Validation` interface that we can use in order to create custom object validation, if some errors occur in the validation process it store them in a `BindingResult` object so you can test for and retrieve validation errors. Let's consider the following validator:

```groovy
package com.jos.dem.springboot.validation.validator

import org.springframework.validation.Validator
import org.springframework.validation.Errors
import org.springframework.stereotype.Component

import com.jos.dem.springboot.validation.command.PersonCommand

@Component
class PersonValidator implements Validator {

  @Override
  boolean supports(Class<?> clazz) {
    PersonCommand.class.equals(clazz)
  }

  @Override
  void validate(Object target, Errors errors) {
    PersonCommand personCommand = (PersonCommand) target
    validateEin(errors, personCommand)
  }

  private void validateEin(Errors errors, PersonCommand command) {
    if(!command.ein.isNumber()){
      errors.rejectValue('ein', 'ein.error.format')
    }
  }

}
```

This class provides validation behaviour implementing `org.springframework.validation.Validator`

* `supports(Class)` Define which class can be validated
* `validate(Object, org.springframework.validation.Errors)` Object validation, if some errors occurs store them in `Errors`
* `validateEin(Errors errors, PersonCommand command)` validates EIN(Employer Identification Number) which should be a digit number.

```groovy
package com.jos.dem.springboot.validation.controller

import static org.springframework.web.bind.annotation.RequestMethod.GET
import static org.springframework.web.bind.annotation.RequestMethod.POST

import javax.validation.Valid

import org.springframework.stereotype.Controller
import org.springframework.web.servlet.ModelAndView
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.ResponseBody
import org.springframework.web.bind.annotation.InitBinder
import org.springframework.web.bind.WebDataBinder
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.validation.BindingResult

import com.jos.dem.springboot.validation.model.Person
import com.jos.dem.springboot.validation.command.Command
import com.jos.dem.springboot.validation.command.PersonCommand
import com.jos.dem.springboot.validation.validator.PersonValidator
import com.jos.dem.springboot.validation.repository.PersonRepository

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@Controller
@RequestMapping('persons/**')
class PersonController {

  @Autowired
  PersonRepository personRepository
  @Autowired
  PersonValidator personValidator

  Logger log = LoggerFactory.getLogger(this.class)

  @InitBinder
  private void initBinder(WebDataBinder binder) {
    binder.addValidators(personValidator)
  }

  @RequestMapping(method=GET)
  ModelAndView getAll(){
    log.info 'Listing all persons'
    ModelAndView modelAndView = new ModelAndView('persons/list')
    List<Person> persons = personRepository.findAll()
    modelAndView.addObject('persons', persons)
    modelAndView
  }

  @RequestMapping(value='create', method=GET)
  ModelAndView create(){
    log.info 'Creating person'
    ModelAndView modelAndView = new ModelAndView('persons/create')
    Command personCommand = new PersonCommand()
    modelAndView.addObject('personCommand', personCommand)
    modelAndView
  }

  @RequestMapping(method=POST)
  ModelAndView save(@Valid PersonCommand personCommand, BindingResult bindingResult){
    log.info "Registering new Person: ${personCommand.nickname}"
    ModelAndView modelAndView = new ModelAndView('persons/list')
    if(bindingResult.hasErrors()){
      modelAndView.setViewName('persons/create')
      modelAndView.addObject('personCommand', personCommand)
      return modelAndView
    }
    Person person = new Person(nickname:personCommand.nickname, email:personCommand.email, ein:personCommand.ein)
    personRepository.save(person)
    List<Person> persons = personRepository.findAll()
    modelAndView.addObject('persons', persons)
    modelAndView
  }

}
```

You can retrieve all the attributes from the form bound to the `PersonCommand` object. In the code, you test for errors, and if so, send the user back to the original form template. In that situation, all the error attributes are displayed.

If all of the user’s attribute are valid, then it redirects the browser to the list persons template.

```html
<html>
<head>
  <link rel="stylesheet" th:href="@{/assets/third-party/bootstrap/dist/css/bootstrap.min.css}" />
</head>
<body>
  <nav class="navbar navbar-inverse navbar-fixed-top">
    <h3><font color="white">Spring Boot</font></h3>
  </nav>
  <div class="jumbotron">
    <div class="container">
      <h1>Welcome!</h1>
      <p>Spring Boot Validator</p>
      <a href="https://github.com/josdem/spring-boot-training" class="btn btn-primary btn-lg" role="button">Learn more</a>
    </div>
  </div>
  <div class="container">
  <form id="create" th:action="@{/persons}" th:object="${personCommand}" method="post">
    <div class="form-group">
      <label class="col-sm-1 col-form-label-lg" for="nickname">Nickname:</label>
      <input class="form-control form-control-lg" type="text" name="nickname" th:field="*{nickname}" placeholder="nickname" id="nickname"/>
      <label th:if="${#fields.hasErrors('nickname')}" th:errors="*{nickname}"></label>
    </div>
    <br/>
    <div class="form-group">
      <label class="col-sm-1 col-form-label-lg" for="email">Email:</label>
      <input class="form-control form-control-lg" type="text" name="email" th:field="*{email}" placeholder="email" id="email"/>
      <label th:if="${#fields.hasErrors('email')}" th:errors="*{email}"></label>
    </div>
    <br/>
    <div class="form-group">
      <label class="col-sm-1 col-form-label-lg" for="ein">EIN:</label>
      <input class="form-control form-control-lg" type="text" name="ein" th:field="*{ein}" placeholder="EIN" id="ein"/>
      <label th:if="${#fields.hasErrors('ein')}" th:errors="*{ein}"></label>
    </div>
    <br/><br/>
    <button class="btn btn-success" id="btn-success" type="submit">Submit</button>
  </form>
  </div>
  <br/><br/><br/>
  <footer>
    <nav class="navbar navbar-inverse navbar-fixed-bottom">
      <a class="navbar-brand" href="https://github.com/josdem/spring-boot-training">josdem 2018</a>
    </nav>
  </footer>
</body>
</html>
```

The page contains a simple form. It is marked as being backed up by the person object that you saw in the GET method in the web controller. This is known as a bean-backed form. You can see them tagged `th:field="{nickname}"`, etc. Next to each field is a secondary element used to show any validation errors.

```groovy
package com.jos.dem.springboot.validation.command

import javax.validation.constraints.Size
import javax.validation.constraints.NotNull
import org.hibernate.validator.constraints.Email

class PersonCommand implements Command {

  @NotNull
  @Size(min=3, max=50)
  String nickname

  @Email
  @NotNull
  @Size(min=1, max=250)
  String email

  @NotNull
  @Size(min=9, max=9)
  String ein

}
```

In the model `PersonCommand` we defined all fields as not null, size min and max and email validation format using hibernate validator.

`Command` is just a Serializable interface

```groovy
package com.jos.dem.springboot.validation.command

import java.io.Serializable

interface Command extends Serializable{}
```

This is the test to cover our custom `PersonValidator`

```groovy
package com.jos.dem.springboot.validation

import org.springframework.validation.Errors

import com.jos.dem.springboot.validation.command.PersonCommand
import com.jos.dem.springboot.validation.validator.PersonValidator

import spock.lang.Specification

class PersonValidatorSpec extends Specification {

  PersonValidator validator = new PersonValidator()

  void "should detect EIN error format"(){
    given:'An EIN format'
      String ein = 'josdem'
    and:'A target and a Error'
      PersonCommand target = Mock(PersonCommand)
      Errors errors = Mock(Errors)
    when:'We validate EIN'
      target.ein >> ein
      validator.validate(target, errors)
    then:'We expect an error added'
      1 * errors.rejectValue('ein','ein.error.format')
  }

  void "should not detect EIN error format"(){
    given:'An EIN format'
      String ein = '123456789'
    and:'A target and a Error'
      PersonCommand target = Mock(PersonCommand)
      Errors errors = Mock(Errors)
    when:'We validate EIN'
      target.ein >> ein
      validator.validate(target, errors)
    then:'We expect an error added'
      0 * errors.rejectValue('ein','ein.error.format')
  }

}
```

Do not forget to add `spock-spring` and `spring-boot-starter-data-jpa` dependencies to your `build.gradle` file. Here is the complete file for you review

```groovy
buildscript {
  ext {
    springBootVersion = '2.0.0.RELEASE'
  }
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  }
}

plugins {
  id 'com.craigburke.bower-installer' version '2.5.1'
}

bower {
  installBase = 'src/main/resources/static/assets/third-party'
  'bootstrap'('3.3.7'){
    source '**'
  }
}

apply plugin: 'groovy'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.validator'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
  mavenCentral()
}


dependencies {
  compile('org.springframework.boot:spring-boot-starter-web')
  compile('org.springframework.boot:spring-boot-starter-thymeleaf')
  compile('org.springframework.boot:spring-boot-starter-data-jpa')
  compile('mysql:mysql-connector-java:5.1.34')
  compile('org.codehaus.groovy:groovy')
  testCompile('org.spockframework:spock-spring:1.1-groovy-2.4')
  testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

To download the project

```bash
git clone https://github.com/josdem/spring-boot-validation.git
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
