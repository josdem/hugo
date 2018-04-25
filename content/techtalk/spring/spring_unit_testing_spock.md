+++
title = "Unit Testing With Spock"
categories = ["techtalk","code","spring framework"]
tags = ["josdem", "techtalks", "programming", "technology","spring framework"]
date = "2016-01-25T18:27:38-06:00"
description = "We are going to test using unit test with Spock the LoggerInterceptor class."
+++

We are going to test using unit test with Spock the LoggerInterceptor class, for more information regarding what this class does, please visit this post: [IP Whitelist Using Interceptors](/techtalk/spring_interceptor)


```groovy
package com.jos.dem.jmailer.interceptor

import javax.servlet.http.HttpServletRequest
import javax.servlet.http.HttpServletResponse

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.web.servlet.HandlerInterceptor
import org.springframework.web.servlet.ModelAndView
import javax.annotation.PostConstruct

import com.jos.dem.jmailer.service.LoggerService
import com.jos.dem.jmailer.constant.ApplicationConstants

class LoggerInterceptor implements HandlerInterceptor {

  @Autowired
  LoggerService loggerService
  @Autowired
  Properties properties

  def whiteList = []

  @PostConstruct
  public void setup(){
    whiteList = properties.getProperty(ApplicationConstants.WHITE_LIST).tokenize(',')
  }

  boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
    def data = [:]
    data.remoteHost = request.remoteHost
    data.timeInMillis = System.currentTimeMillis()
    data.method = request.method
    data.requestURL = request.requestURL
    data.parameters = request.parameterMap

    if(!whiteList.contains(request.remoteHost)){
      data.warn = "UNAUTORIZED IP was detected in attempt to access to resource"
      loggerService.notifyRequest(data)
      return false
    }

    loggerService.notifyRequest(data)
    return true
  }


  void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {
  }


  void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
  }

}
```

Regarding LogginInterceptorSpec we can consider the following aspects:

* Any Spock test must extend Specification
* Setup method is executed before any test defined in our test class
* We are defining mock objects using Mock annotation
* In the setup method we are assigning our mocks

```groovy
package com.jos.dem.jmailer.interceptor

import spock.lang.Specification

import javax.servlet.http.HttpServletRequest
import javax.servlet.http.HttpServletResponse

import com.jos.dem.jmailer.service.LoggerService

class LoggerInterceptorSpec extends Specification {

  def interceptor = new LoggerInterceptor()

  def request = Mock(HttpServletRequest)
  def response = Mock(HttpServletResponse)
  def loggerService = Mock(LoggerService)
  def properties = new Properties()

  def setup(){
    properties.setProperty('white.list','127.0.0.1')
    interceptor.loggerService = loggerService
    interceptor.properties = properties
    interceptor.setup()
  }

  void "should accept localhost"(){
    given:"An ip address"
      String localhost = '127.0.0.1'
    when:"We preHandle request"
     request.remoteHost >> localhost
     def result = interceptor.preHandle(request, response, new Object())
    then:"We expect access"
     result
     1 * loggerService.notifyRequest(_ as Map)
  }

  void "should not accept strangers"(){
    given:"An ip address"
      String localhost = '127.0.0.1'
    when:"We preHandle request"
     request.remoteHost >> '189.217.63.188'
     def result = interceptor.preHandle(request, response, new Object())
    then:"We expect access denied"
     !result
     1 * loggerService.notifyRequest(_ as Map)
  }
}
```

Regarding our first test method we should read as follow:

* Given an ip address as String
* When `request.remoteHost` return '127.0.0.1' // We are defining our mock behavior
* `def result = interceptor.preHandle(request, response, new Object())` // In this line we are calling our testing method
* Then we expect the result is `true`
* `loggerService.notifyRequest(_ as Map)` // This method is called once with a Map as parameter

For more information how to use Spock [Spock Framework Reference Documentation](http://spockframework.github.io/spock/docs/1.0/index.html)

To download the project:

```bash
git clone https://github.com/josdem/jmailer-bootstrap.git
git fetch
git checkout feature/testing
```

To run this test:

```bash
gradle -Dtest.single=LoggerInterceptorSpec :web:test
```

[Return to the main article](/techtalk/spring)

