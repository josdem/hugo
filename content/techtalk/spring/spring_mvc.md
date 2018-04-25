+++
title = "Basic Spring MVC with Bootstrap"
categories = ["techtalk", "code","spring framework"]
tags = ["josdem", "techtalks", "programming", "technology","spring framework"]
date = "2015-10-12T20:26:12-05:00"
description = "Creating a simple Spring MVC application using Bootstrap and XML configuration"
+++

## Technologies

* Gradle 2.7
* Spring 4.1.7
* Java 8
* Groovy 2.4.5
* Bootstrap 3.3.4
* XML configuration

**build.gradle**

```bash
apply plugin: 'groovy'
apply plugin: 'war'
apply plugin: 'jetty'

sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
    mavenLocal()
    mavenCentral()
}

def springVersion = '4.1.7.RELEASE'
def groovyVersion = '2.4.5'
def aspectjVersion = '1.8.7'
def currentEnvironment = project.hasProperty("environment")?environment:"development"

dependencies {

  compile "org.springframework:spring-webmvc:${springVersion}"
  compile "org.codehaus.groovy:groovy:${groovyVersion}"
  compile 'log4j:log4j:1.2.17'

}

jettyRun{
  contextPath = "jmailer"
  httpPort = 8080
}

jettyRunWar{
  contextPath = "jmailer"
  httpPort = 8080
}

println "Setting environment to: ${currentEnvironment}"

task settingLog4jProperties(type:Copy){
  from "${System.getProperty('user.home')}/.jmailer/log4j-${currentEnvironment}.properties"
  into "src/main/resources/"
  rename { String fileName -> fileName.replace("-${currentEnvironment}", '') }
}

processResources.dependsOn "settingLog4jProperties"
```

**Spring controller**

```groovy
package com.jos.dem.jmailer.controller

import java.util.Map

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.stereotype.Controller
import org.springframework.web.bind.annotation.RequestMapping

import static org.springframework.web.bind.annotation.RequestMethod.GET

import com.jos.dem.jmailer.service.EmailerService

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Controller
class EmailerController {

  @Autowired
  EmailerService emailerService

  Log log = LogFactory.getLog(this.class)

  @RequestMapping(value = '/', method = GET)
  String index(Map<String, Object> model) {
    log.debug 'Calling index'
    model.put('title', emailerService.getTitle('World!'))
    model.put('msg', emailerService.getDescription())

    return 'index'
  }
}
```

**Service**

```groovy
package com.jos.dem.jmailer.service

import org.springframework.stereotype.Service
import org.springframework.util.StringUtils

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Service
class EmailerService {

  Log log = LogFactory.getLog(this.class)

  String getTitle(String name) {
    log.debug "GETTING title with name : ${name}"
    "Hello ${name}"
  }

  String getDescription() {
    log.debug "GETTING description"
    "Jmailer is a service for delivering emails"
  }

}
```

**Views – JSP + JSTL + bootstrap. A simple JSP page to display the model, and includes the static resources like css and js.**

File: /WEB-INF/views/jsp/index.jsp

```xml
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE html>
<html lang="en">
<head>
<title>Jmailer</title>

<spring:url value="/resources/core/css/jmailer.css" var="coreCss" />
<spring:url value="/resources/core/css/bootstrap.min.css" var="bootstrapCss" />
<link href="${bootstrapCss}" rel="stylesheet" />
<link href="${coreCss}" rel="stylesheet" />
</head>

<nav class="navbar navbar-inverse navbar-fixed-top">
  <div class="container">
    <div class="navbar-header">
      <a class="navbar-brand" href="#">Email deliver service</a>
    </div>
  </div>
</nav>

<div class="jumbotron">
  <div class="container">
    <h1>${title}</h1>
    <p>
      <c:if test="${not empty msg}">
        About: ${msg}
      </c:if>
    </p>
    <p>
      <a class="btn btn-primary btn-lg" href="#" role="button">Learn more</a>
    </p>
  </div>
</div>

<div class="container">

  <div class="row">
    <div class="col-md-4">
      <h2>Heading</h2>
      <p>Text</p>
      <p>
        <a class="btn btn-default" href="#" role="button">Action</a>
      </p>
    </div>
  </div>


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

**Spring XML Configuration**

File: /WEB-INF/dispatcher-servlet.xml

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

  <context:component-scan base-package="com.jos.dem.jmailer" />


  <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
    <property name="prefix" value="/WEB-INF/views/jsp/" />
    <property name="suffix" value=".jsp" />
  </bean>

  <mvc:resources mapping="/resources/**" location="/resources/" />

  <mvc:annotation-driven />

</beans>

```

File: /WEB-INF/web.xml

```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
  version="3.0">

  <display-name>Jmailer</display-name>
  <description>Emailer deliver service</description>

  <servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
  </servlet>

  <servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

</web-app>
```

In your home, create a folder named .jmailer and add this log4j file

File: log4j-development.properties

```bash
#
# The logging properties used for eclipse testing, We want to see INFO output on the console.
#
log4j.rootLogger=out

log4j.logger.com.jos.dem=DEBUG,out
log4j.logger.org.springframework=INFO,out
log4j.logger.org.springframework.transaction=DEBUG
log4j.logger.org.springframework.jmx=ERROR,out
log4j.logger.org.springframework.aop=DEBUG
log4j.logger.org.hibernate=ERROR,out
log4j.logger.org.apache.commons.beanutils=ERROR,out
log4j.logger.org.displaytag=ERROR,out
log4j.logger.net.sf=ERROR,out

#Ensure the logs don't add to each other
log4j.additivity.com.tim.one=false
log4j.additivity.org.springframework=false
log4j.additivity.org.springframework.jmx=false
log4j.additivity.org.hibernate=false
log4j.additivity.org.apache.commons.beanutils=false
log4j.additivity.org.displaytag=false
log4j.additivity.net.sf=false

log4j.appender.out=org.apache.log4j.ConsoleAppender
log4j.appender.out.layout=org.apache.log4j.PatternLayout
log4j.appender.out.layout.ConversionPattern=%d %5p [%t] (%F:%L) - %m%n
```

In order to run go to your home directory and type:

```bash
gradle jettyRun
```

Go in your browser to: http://localhost:8080/jmailer/

To download the project

```bash
git clone https://github.com/josdem/jmailer-bootstrap.git
git fetch
git checkout setup
```

[Return to the main article](/techtalk/spring)
