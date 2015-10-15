+++
date = "2015-10-14T21:41:43-05:00"
draft = true
title = "JMS Logger with ActiveMQ"

+++

The main beneffit to log our application with JMS is to use asynchronous logging, meaning services creates some log messages, sends them across the network to a JMS provider and then the receiver store that message in our log system.

First we need to set our logger service structure

```groovy
package com.jos.dem.jmailer.service

interface LoggerService {
  void notifyRequest(def requestParams)
}
```

Logger service implementation

```groovy
package com.jos.dem.jmailer.service.impl

import javax.jms.Message
import javax.jms.ObjectMessage
import javax.jms.Session
import javax.jms.Destination
import javax.jms.JMSException

import org.springframework.jms.core.JmsTemplate
import org.springframework.jms.core.MessageCreator
import org.springframework.stereotype.Service
import org.springframework.beans.factory.annotation.Autowired

import com.jos.dem.jmailer.service.LoggerService

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Service
class LoggerServiceImpl implements LoggerService {

  @Autowired
  JmsTemplate template

  @Autowired
  Destination logger

  Log log = LogFactory.getLog(getClass())

  void notifyRequest(requestParams) {
    log.info "CALLING Notifying"

      template.send(logger, new MessageCreator() {
        public Message createMessage(Session session) throws JMSException {
          ObjectMessage message = session.createObjectMessage()
          message.setObject(requestParams)
          message
        }
      })
  }

}
```

In our logger service implementation we create an message and using an JMS template we deliver our message to the destination.

The receiver should be an MessageListener implementation

```groovy
package com.jos.dem.jmailer.messengine

import javax.jms.Message
import javax.jms.MessageListener
import javax.jms.ObjectMessage

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.stereotype.Service


@Service
class LoggerMessageListener implements MessageListener {

  Log log = LogFactory.getLog(getClass())

  def void onMessage(Message message) {
    def data =  ((ObjectMessage) message).getObject()
    String dataToString = data.collect { k, v -> v }.join(';')
    log.info dataToString
  }

}
```

In this class we receive our message, and using collect() method we iterate over message and transform each element in an string delimited by ;

Now, the idea to use this structure is to catch all request from our client from the controller using an interceptor as follow:

```groovy
package com.jos.dem.jmailer.interceptor

import javax.servlet.http.HttpServletRequest
import javax.servlet.http.HttpServletResponse

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.web.servlet.HandlerInterceptor
import org.springframework.web.servlet.ModelAndView

import com.jos.dem.jmailer.service.LoggerService

class LoggerInterceptor implements HandlerInterceptor {

  @Autowired
  LoggerService loggerService

  boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
    def data = [:]
    data.remoteHost = request.remoteHost
    data.timeInMillis = System.currentTimeMillis()
    data.method = request.method
    data.requestURL = request.requestURL
    data.parameters = request.parameterMap
    loggerService.notifyRequest(data)
    return true
  }


  void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) {
  }


  void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) {
  }

}
```

We need to define our interceptor in the dispatcher servlet.

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
  <import resource="classpath:/jms-appctx.xml" />
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

</beans>
```

Finally we need to declare our jms context, and don't forget to import it to the dispatcher servler.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:p="http://www.springframework.org/schema/p"
  xmlns:jms="http://www.springframework.org/schema/jms"
  xmlns:amq="http://activemq.apache.org/schema/core"
  xsi:schemaLocation="http://www.springframework.org/schema/jms http://www.springframework.org/schema/jms/spring-jms.xsd
  http://activemq.apache.org/schema/core http://activemq.apache.org/schema/core/activemq-core.xsd
  http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
  http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

  <!--  Embedded ActiveMQ Broker -->
  <amq:broker id="broker" useJmx="false" persistent="false">
    <amq:transportConnectors>
      <amq:transportConnector uri="tcp://localhost:0" />
    </amq:transportConnectors>
  </amq:broker>

  <!--  ActiveMQ Destination  -->
  <amq:queue id="logger" physicalName="loggerMessageListener" />

  <!-- JMS ConnectionFactory to use, configuring the embedded broker using XML -->
  <amq:connectionFactory id="jmsFactory" brokerURL="vm://localhost" />

  <!-- JMS Producer Configuration -->
  <bean id="jmsProducerConnectionFactory"
    class="org.springframework.jms.connection.SingleConnectionFactory"
    depends-on="broker"
    p:targetConnectionFactory-ref="jmsFactory" />

  <bean id="jmsProducerTemplate" class="org.springframework.jms.core.JmsTemplate"
    p:connectionFactory-ref="jmsProducerConnectionFactory" />

  <!-- JMS Consumer Configuration -->
  <bean id="jmsConsumerConnectionFactory"
    class="org.springframework.jms.connection.SingleConnectionFactory"
    depends-on="broker"
    p:targetConnectionFactory-ref="jmsFactory" />

  <jms:listener-container container-type="default"
    connection-factory="jmsConsumerConnectionFactory"
    acknowledge="auto">
    <jms:listener destination="loggerMessageListener" ref="loggerMessageListener" />
  </jms:listener-container>

</beans>
```

To download the project

```bash
git clone git@github.com:josdem/jmailer-bootstrap.git
git fetch
git checkout feature/jms-logger
```

[Return to the main article](/techtalk/spring)

