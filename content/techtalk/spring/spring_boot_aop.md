+++
date = "2015-11-29T21:25:06-06:00"
draft = true
title = "Spring Boot AOP"

+++

Apply same logic in several points in your application is called cross cutting concerns, and in Spring we can implement it using AOP, some basics aspects are:

* Advice: Action taken. Different types of advice include “around,” “before”, “after”, "AfterReturning" and "AfterThrowing" advice.
* Pointcut: Where the action should be applied.
* Aspect: Is an advice and pointcut combination

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
import org.slf4j.Logger
import org.slf4j.LoggerFactory

import com.jos.dem.jmailer.exception.BusinessException

@Aspect
@Component
class AfterThrowingAdvice {

  Logger log = LoggerFactory.getLogger(this.class)

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
import org.slf4j.Logger
import org.slf4j.LoggerFactory

import com.jos.dem.jmailer.exception.EmailerException

@Service
class EmailerService {

  Logger log = LoggerFactory.getLogger(this.class)

  def sendEmail(){
    log.debug 'Sending email'
    throw new EmailerException()
  }

}
```

As you can see at EmailerService in the sendEmail() method we are throwing an EmailerException so our Advice will be called.

Here is the EmailerController and a quick view how this controller is calling to the EmailerService.

```groovy
package com.jos.dem.jmailer.controller

import org.springframework.web.bind.annotation.RestController
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.beans.factory.annotation.Autowired

import com.jos.dem.jmailer.service.EmailerService

@RestController
class EmailerController {

  @Autowired
  EmailerService emailerService

  @RequestMapping("/")
  String index() {
    emailerService.sendEmail()
  }

}
```

To get this work, you only need to add aop dependency in your build.gradle

```bash
compile 'org.springframework.boot:spring-boot-starter-aop'
```

To download the project:

```bash
git clone https://github.com/josdem/jmailer-spring-boot.git
git fetch
git checkout feature/aop
```

[<img src="https://farm1.staticflickr.com/650/23150308934_3864b02cec_b.jpg">](https://youtu.be/3xrRIesGneg)

[Return to the main article](/techtalk/spring)
