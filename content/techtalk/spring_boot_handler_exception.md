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

To download the project:

```bash
git clone git@github.com:josdem/jmailer-spring-boot.git
git fetch
git checkout feature/handler-exception
```

[Return to the main article](/techtalk/spring)
