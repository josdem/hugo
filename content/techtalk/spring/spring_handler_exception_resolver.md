+++
date = "2015-11-12T20:28:41-06:00"
draft = true
title = "Handler Exception Resolver"

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

File: $JMAILER_HOME/web/src/main/webapp/WEB-INF/jsp/error.jsp

```html
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html lang="en">
<head>
<title>Jmailer</title>

<spring:url value="/resources/core/css/jmailer.css" var="coreCss" />
<spring:url value="/resources/core/css/bootstrap.min.css" var="bootstrapCss" />
<link href="${bootstrapCss}" rel="stylesheet" />
<link href="${coreCss}" rel="stylesheet" />
</head>

<body>

  Error: <c:out value="${data.message}"/>

  <hr>
  <footer>
    <p>&copy; josdem 2015</p>
  </footer>
</div>

<spring:url value="/resources/core/css/hello.js" var="coreJs" />
<spring:url value="/resources/core/css/bootstrap.min.js" var="bootstrapJs" />

<script src="${coreJs}"></script>
<script src="${bootstrapJs}"></script>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.11.2/jquery.min.js"></script>

</body>
</html>
```

This is a basic html file wtih a message to the user, the HandlerException message

Finally we specify our HandlerException class in our servlet xml.

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:mvc="http://www.springframework.org/schema/mvc"
  xsi:schemaLocation="
  http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans.xsd
  http://www.springframework.org/schema/mvc
  http://www.springframework.org/schema/mvc/spring-mvc.xsd
  http://www.springframework.org/schema/context
  http://www.springframework.org/schema/context/spring-context.xsd ">

  <import resource="classpath:/aop-appctx.xml" />
  <import resource="classpath:/jms-appctx.xml" />
  <import resource="classpath:/formatter-appctx.xml" />
  <import resource="classpath:/mail-appctx.xml" />
  <context:component-scan base-package="com.jos.dem.jmailer" />

  <mvc:interceptors>
    <bean class="com.jos.dem.jmailer.interceptor.LoggerInterceptor" />
  </mvc:interceptors>

  <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    <property name="prefix" value="/WEB-INF/views/jsp/" />
    <property name="suffix" value=".jsp" />
  </bean>

  <mvc:resources mapping="/resources/**" location="/resources/" />

  <mvc:annotation-driven />

  <bean class="com.jos.dem.jmailer.controller.HandlerException" />

</beans>
```

To download the project

```bash
git clone https://github.com/josdem/jmailer-bootstrap.git
git fetch
git checkout feature/java-mail
```

[Return to the main article](/techtalk/spring)

