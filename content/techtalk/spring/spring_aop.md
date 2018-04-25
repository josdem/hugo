+++
title = "Aspect Oriented Programming"
categories = ["techtalk", "code","spring framework"]
tags = ["josdem", "techtalks", "programming", "technology","spring framework"]
date = "2015-10-13T22:00:48-05:00"
description = "Apply same logic in several points in your application is called cross cutting concerns, and in Spring we can implement it using AOP"
+++

Apply same logic in several points in your application is called cross cutting concerns, and in Spring we can implement it using AOP, some basics aspects are:

  * Advice: Action taken. Different types of advice include "around," "before" and "after" advice.
  * Pointcut: Where the action should be applied.

In this example we are going to define an AfterThrowingAdvice which means an action is executed after some exception has been thrown, so first we need to define a general exception in our application.

```groovy
package com.jos.dem.jmailer.exception

import org.springframework.core.NestedRuntimeException

class BusinessException extends NestedRuntimeException {

  BusinessException(String msg){
    super(msg)
  }

  BusinessException(String msg, Throwable cause) {
    super(msg, cause)
  }

}

```

BusinessException is a general exception in our application defined to react in the same way to the specific exception, some specific exception could be an EmailerException.

```groovy
package com.jos.dem.jmailer.exception

import java.lang.RuntimeException

class EmailerException extends RuntimeException {

  @Override
  String getMessage() {
    "Emailer exception"
  }

}
```

So our advice look like this:

```groovy
package com.jos.dem.jmailer.advice

import org.aspectj.lang.annotation.AfterThrowing
import org.aspectj.lang.annotation.Aspect
import org.springframework.stereotype.Component

import com.jos.dem.jmailer.exception.BusinessException

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Aspect
@Component
class AfterThrowingAdvice {

  Log log = LogFactory.getLog(this.class)

  @AfterThrowing(pointcut = "execution(* com.jos.dem.jmailer.service..**.*(..))", throwing = "ex")
  public void doRecoveryActions(RuntimeException ex) {
    log.info "Wrapping exception ${ex}"
    throw new BusinessException(ex.getMessage(), ex)
  }

}
```

**Explanation**

* An aspect is defined with the @Aspect annotation
* A bean managed by the spring context is defined with the @Component annotation
* A pointcut is defined as all methods in package com.jos.dem.jmailer.service with any parameter
* The action defined in the advice is thrown an BusinessException

So if in any service we throw an Exception, actually a RuntimeException our advice logic is executed.


```groovy
package com.jos.dem.jmailer.service

import org.springframework.stereotype.Service
import org.springframework.util.StringUtils
import com.jos.dem.jmailer.exception.EmailerException

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

  def sendEmail(){
    log.debug 'Sending email'
    throw new EmailerException()
  }

}
```

As you can see at EmailerService in the sendEmail() method we are throwing an EmailerException so our Advice will be called.

In order to define our AOP in spring we need an aop context file in the following path: src/main/resources/aop-appctx.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:aop="http://www.springframework.org/schema/aop"
  xsi:schemaLocation="http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd
  http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
  http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

  <aop:aspectj-autoproxy/>

  <context:component-scan base-package="com.jos.dem.jmailer.advice"/>

</beans>

```

Finally we need to import our aop-appctx.xml in our servlet as well.

File: src/main/webapp/WEB-INF/dispatcher-servlet.xml

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

To download the project

```bash
git clone https://github.com/josdem/jmailer-bootstrap.git
git fetch
git checkout feature/aop-business-exception
```

[Return to the main article](/techtalk/spring)
