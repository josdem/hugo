+++
date = "2017-01-22T21:19:21-06:00"
title = "Spring Boot Validation Testing"
tags = ["josdem","techtalks","programming","technology"]
categories = ["techtalk","code"]

+++

This time I will show you how create a unit testing using [Spock](http://spockframework.org/), Spock is a testing and specification framework for Java and Groovy applications it is highly expressive and easy to use. First you need to add `Spock dependency` in your `build.gradle`

```groovy
testCompile 'org.spockframework:spock-spring'
```

This is the class we are going to test:

```groovy
package com.jos.dem.vetlog.validator

import org.springframework.validation.Validator
import org.springframework.validation.Errors
import org.springframework.stereotype.Component
import org.springframework.beans.factory.annotation.Autowired

import com.jos.dem.vetlog.service.LocaleService
import com.jos.dem.vetlog.command.UserCommand

@Component
class UserValidator implements Validator {

  @Autowired
  LocaleService localeService

  @Override
  boolean supports(Class<?> clazz) {
    UserCommand.class.equals(clazz)
  }

  @Override
  void validate(Object target, Errors errors) {
    UserCommand UserCommand = (UserCommand) target
    validatePasswords(errors, UserCommand)
  }

  def validatePasswords(Errors errors, UserCommand command) {
    if (!command.password.equals(command.passwordConfirmation)){
      errors.reject('password', localeService.getMessage('user.validation.password.equals'))
    }
  }

}
```

Please read this [post](/techtalk/spring/spring_boot_validation) to get familiar with this previous class and validators.

```groovy
package com.jos.dem.vetlog

import org.springframework.validation.Errors

import spock.lang.Specification

import com.jos.dem.vetlog.command.Command
import com.jos.dem.vetlog.command.UserCommand
import com.jos.dem.vetlog.service.LocaleService
import com.jos.dem.vetlog.validator.UserValidator

class UserValidatorSpec extends Specification {

  UserValidator validator = new UserValidator()
  Errors errors = Mock(Errors)
  LocaleService localeService = Mock(LocaleService)

  def setup(){
    validator.localeService = localeService
  }

  void "should not validate an user command since passwords are not equals"(){
    given:"A user command"
      Command command = new UserCommand(username:'josdem',password:'password', passwordConfirmation:'p4ssword', name:'josdem',lastname:'lastname',email:'josdem@email.com')
    when:"We validate passwords"
      localeService.getMessage('user.validation.password.equals') >> 'The passwords are not equals'
      validator.validate(command, errors)
    then:"We expect valiation failed"
    1 * errors.reject('password', 'The passwords are not equals')
  }

  void "should vaidate an user command"(){
    given:"A user command"
      Command command = new UserCommand(username:'josdem',password:'password', passwordConfirmation:'password', name:'josdem',lastname:'lastname',email:'josdem@email.com')
    when:"We validate passwords"
      localeService.getMessage(_ as String) >> 'Passwords are bad formed'
      validator.validate(command, errors)
    then:"We expect everything is going to be all right"
    0 * errors.reject('password', _ as String)
  }

  void "should accept characters and numbers in password"(){
    given:"A user command"
      Command command = new UserCommand(username:'josdem',password:'p4ssword', passwordConfirmation:'p4ssword', name:'josdem',lastname:'lastname',email:'josdem@email.com')
    when:"We validate passwords"
      localeService.getMessage(_ as String) >> 'Passwords are bad formed'
      validator.validate(command, errors)
    then:"We expect everything is going to be all right"
    0 * errors.reject('password', _ as String)
  }

  void "should accept dash character in password"(){
    given:"A user command"
      Command command = new UserCommand(username:'josdem',password:'pa-4ssword', passwordConfirmation:'pa-4ssword', name:'josdem',lastname:'lastname',email:'josdem@email.com')
    when:"We validate passwords"
      localeService.getMessage(_ as String) >> 'Passwords are bad formed'
      validator.validate(command, errors)
    then:"We expect everything is going to be all right"
    0 * errors.reject('password', _ as String)
  }

  void "should accept underscore character in password"(){
    given:"A user command"
      Command command = new UserCommand(username:'josdem',password:'pa_4ssword', passwordConfirmation:'pa_4ssword', name:'josdem',lastname:'lastname',email:'josdem@email.com')
    when:"We validate passwords"
      localeService.getMessage(_ as String) >> 'Passwords are bad formed'
      validator.validate(command, errors)
    then:"We expect everything is going to be all right"
    0 * errors.reject('password', _ as String)
  }

  void "should accept dot character in password"(){
    given:"A user command"
      Command command = new UserCommand(username:'josdem',password:'pa.4ssword', passwordConfirmation:'pa.4ssword', name:'josdem',lastname:'lastname',email:'josdem@email.com')
    when:"We validate passwords"
      localeService.getMessage(_ as String) >> 'Passwords are bad formed'
      validator.validate(command, errors)
    then:"We expect everything is going to be all right"
    0 * errors.reject('password', _ as String)
  }

}
```

This are some important topics about the previous test class:

1. We are defining `org.springframework.validation.Errors` as a Mock
2. Also our internationalization class `LocaleService` is a Mock
3. In the `setup` method we are assigning the mocking `LocaleService` to the `UserValidator` class
4. Spock provides a convenient way to verify the number of times a mock method is called, in this case we are expecting that the method `errors.reject('password', 'The passwords are not equals')` was called once.
5. When the method from our mock is called here `localeService.getMessage('user.validation.password.equals')` we will return `The passwords are not equals` String
6. Spock provides a lenient type of argument matching in a mock method call is the underscore, which will match anything that is passed into it but a String. `localeService.getMessage(_ as String) >> 'Passwords are bad formed'`

That's it, we did add some other tests to verify our functionality

To download the project

```bash
git clone https://github.com/josdem/vetlog-spring-boot.git
git fetch
git checkout feature/8
```

To run the project, please read the [wiki](https://github.com/josdem/vetlog-spring-boot/wiki/YAML%20File) and execute this command:

```bash
gradle bootRun -Dspring.config.location=$HOME/.vetlog-spring-boot/application-development.yml
```

[Return to the main article](/techtalk/spring)

