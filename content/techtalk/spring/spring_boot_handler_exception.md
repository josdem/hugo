+++
date = "2015-11-29T21:34:44-06:00"
draft = true
title = "Spring Boot Handler Exception"

+++

There are several reasons why we should manage exceptions such as:

* Customize an error page
* Business logic
* Reduce logic

The common way to manage exceptions in Spring is using an HandlerExceptionResolver class, in this example I will show you how to manage an generic exception in our code.

```groovy
package com.jos.dem.jmailer.controller

import org.springframework.web.servlet.HandlerExceptionResolver
import org.springframework.http.HttpStatus
import javax.servlet.http.HttpServletRequest
import javax.servlet.http.HttpServletResponse
import org.springframework.stereotype.Component
import org.springframework.http.ResponseEntity
import org.springframework.web.servlet.ModelAndView

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Component
class HandlerException implements HandlerExceptionResolver {

  Log log = LogFactory.getLog(this.class)

  ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex){
    log.info ex.message
    def data = [:]
    data.message = ex.message
    ModelAndView modelAndView = new ModelAndView("error")
    modelAndView.addObject("data", data)
    modelAndView
  }
}
```

Here we have an HandlerException class that implements HandlerExceptionResolver, and a resolveException method who receives exception, render to the error page and send an data as a Map with some information we want user sees.

Our error page looks like this:

File: $PROJECT_HOME/src/main/resources/templates/error.html

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org" xmlns:layout="http://www.ultraq.net.nz/web/thymeleaf/layout">
  <head>
    <title>Jmailer Error page</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
  </head>
  <body>
    <div th:if="${data}">
      <p th:text="'Error: ' + ${data.message}" />
    </div>
  </body>
</html>
```

To get this work, you only need to add Thymeleaf dependency to your build.gradle.

```groovy
compile 'org.springframework.boot:spring-boot-starter-thymeleaf'
```

Another common way to handle exceptions is to use `@ExceptionHandler` annotation in a method in a controller, I normally get this option when I'm builing an API.

```groovy
package com.jos.dem.jmailer.controller

import static org.springframework.web.bind.annotation.RequestMethod.POST

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.beans.factory.annotation.Value
import org.springframework.web.bind.annotation.RestController
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.web.bind.annotation.ExceptionHandler
import org.springframework.http.ResponseEntity
import org.springframework.http.HttpStatus
import org.slf4j.Logger
import org.slf4j.LoggerFactory

import com.jos.dem.jmailer.service.EmailerService
import com.jos.dem.jmailer.command.MessageCommand
import com.jos.dem.jmailer.exception.BusinessException

@RestController
class EmailerController {

  @Autowired
  EmailerService emailerService

  @Value('${email.redirect}')
  String redirectUrl

  Logger logger = LoggerFactory.getLogger(this.class)

  @RequestMapping(method = POST, value = "/message", consumes="application/json")
  ResponseEntity<String> message(@RequestBody MessageCommand command) {
    logger.info "Sending contact email: ${command.email}"
    emailerService.sendEmail(command)
    new ResponseEntity<String>("OK", HttpStatus.OK)
  }

  @ResponseStatus(value=HttpStatus.UNAUTHORIZED, reason="Unauthorized")
  @ExceptionHandler(BusinessException.class)
  ResponseEntity<String> handleException(BusinessException be){
    return new ResponseEntity<String>("Unauthorized", HttpStatus.UNAUTHORIZED)
  }

}

```

That's it, when a BusinessException occurs the method handleException will respond with a 401 `UNAUTHORIZED` to the client. In this way we avoid to use a `try` `catch` estructure to handle exceptions.

To download the project:

```bash
git clone git@github.com:josdem/jmailer-spring-boot.git
git fetch
git checkout feature/handler-exception
```

[Return to the main article](/techtalk/spring)
