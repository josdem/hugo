+++
date = "2015-11-21T21:41:45-06:00"
draft = true
title = "Spring Interceptor"

+++

Spring MVC provides a powerful mechanism to intercept an http request, provides a way to define special classes called Interceptors that gets called before and after a request is served.

We're going to use inteceptors to get request before passing to the controller to catch IP not autorized to access to the services.

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
      return false;
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

We are using here preHandle method to receive client request, then we fill data which is a map with specific information we want to store from the request. Finally we return true if is an allowed IP or false otherwise.

When we are using logs it ts important several factors such as level, file and locations. This are levels we can use in log4j

* TRACE
* DEBUG
* INFO
* WARN
* ERROR
* FATAL

And that is a precedence factor as well. If we want to controll file and locations we can use and RollingAppender as follow

```bash
#
# The logging properties used for eclipse testing, We want to see INFO output on the console.

log4j.rootLogger=WARN, RollingAppender
log4j.logger.com.jos.dem=WARN, out
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

#Emailer request log
log4j.appender.RollingAppender=org.apache.log4j.DailyRollingFileAppender
log4j.appender.RollingAppender.File=/home/josdem/.jmailer/requests.log
log4j.appender.RollingAppender.DatePattern='.'yyyy-MM-dd
log4j.appender.RollingAppender.layout=org.apache.log4j.PatternLayout
log4j.appender.RollingAppender.layout.ConversionPattern=%m%n
```

Here we are defining where to store our application output, in some specific file and its location.

Finally, we change our logger message to warn level.

```groovy
package com.jos.dem.jmailer.messengine

import javax.jms.Message
import javax.jms.MessageListener
import javax.jms.ObjectMessage

import org.springframework.stereotype.Service
import org.springframework.beans.factory.annotation.Autowired

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Service
class LoggerMessageListener implements MessageListener {

  Log log = LogFactory.getLog(getClass())

  def void onMessage(Message message) {
    def data =  ((ObjectMessage) message).getObject()
    String dataToString = data.collect { k, v -> v }.join(';')
    log.warn dataToString
  }

}
```

To download the project

```bash
git clone git@github.com:josdem/jmailer-bootstrap.git
git fetch
git checkout feature/interceptor
```

[Return to the main article](/techtalk/spring)
