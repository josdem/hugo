+++
tags = ["josdem","techtalks","programming","technology", "spring boot jms", "java message service", "spring boot"]
date = "2018-02-26T09:08:25-06:00"
title = "Spring Boot JMS"
categories = ["techtalk","code"]
+++

The Java Message Service is an API for sending messages between two or more clients. It is an implementation to [Producer-Consumer Design Pattern](https://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem). This technique is usually implemented when you have time consuming process and you need to avoid client waiting time. To put this in context let’s think about a scenario where we could use it. The first thing that comes to my mind is an email delivery process. Sending an email consumes too much time and we can put email delivery as a task in a queue, so we can continue with our business flow without force to the client to wait until its email is deliver.

In this example I will show you how to use JMS in a Spring Boot application. **NOTE:** If you need to know what tools you need to have installed in yout computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot)

Then execute this command in your terminal.

```bash
spring init --dependencies=web --language=groovy --build=gradle spring-boot-jms
```


This is the `build.gradle file` generated:

```groovy
buildscript {
	ext {
		springBootVersion = '1.5.10.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'groovy'
apply plugin: 'org.springframework.boot'

group = 'com.jos.dem.springboot.jms'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}

dependencies {
	compile('org.springframework.boot:spring-boot-starter-web')
	compile('org.codehaus.groovy:groovy')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}
```


First, lets create a `MessageService` to deliver messages to the queue.

```groovy
package com.jos.dem.springboot.jms.service

import com.jos.dem.springboot.jms.command.Command

interface MessageService {

  void sendMessage(final Command command)

}
```

This is our `MessageServiceImpl` implementation class:

```groovy
package com.jos.dem.springboot.jms.service.impl

import javax.jms.JMSException
import javax.jms.ObjectMessage
import javax.jms.Message
import javax.jms.Session

import org.springframework.stereotype.Service
import org.springframework.jms.core.JmsTemplate
import org.springframework.jms.core.MessageCreator
import org.springframework.jms.annotation.EnableJms
import org.springframework.beans.factory.annotation.Autowired

import com.jos.dem.springboot.jms.command.Command
import com.jos.dem.springboot.jms.service.MessageService

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@Service
@EnableJms
class MessageServiceImpl implements MessageService {

  @Autowired
  JmsTemplate jmsTemplate

  Logger log = LoggerFactory.getLogger(this.class)

  void sendMessage(final Command command) {
    MessageCreator messageCreator = new MessageCreator() {

      @Override
      public Message createMessage(Session session) throws JMSException {
        ObjectMessage message = session.createObjectMessage()
        message.setObject(command)
        return message
      }
    }

    log.info 'Sending message'
    jmsTemplate.send("destination", messageCreator)
  }

}
```

`Command` is just a serializable object and it is common for me to use an interface instead and specific POJO.

```groovy
package com.jos.dem.springboot.jms.command

import java.io.Serializable

interface Command extends Serializable {}
```

And this is the message object we are going to send.

```groovy
package com.jos.dem.springboot.jms.command

class PersonCommand implements Command {
	String nickname
	String email
}
```

Now we have all entities we need to set in order to send a message, next we need to specify the entities to receive and process messages

```groovy
package com.jos.dem.springboot.jms.messengine

import javax.jms.Message
import javax.jms.ObjectMessage

import org.springframework.stereotype.Component
import org.springframework.jms.annotation.JmsListener
import org.springframework.beans.factory.annotation.Autowired

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@Component
class MessageListener {

  Logger log = LoggerFactory.getLogger(this.class)

  @JmsListener(destination = "destination", containerFactory = "myJmsContainerFactory")
  public void receiveMessage(Message message) {
    Object command =  ((ObjectMessage) message).getObject()
    log.info "Message Received ${command.dump()}"    
  }

}
```

As you can see `JmsTemplate` is sending a message to the `destination` and `@JmsListener` is waiting for new messages from `destination`, the another important part in this puzzle is the JMS container called `myJmsContainerFactory` who is defined in our Spring Boot application class as a bean.

```groovy
package com.jos.dem.springboot.jms

import javax.jms.ConnectionFactory

import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.context.annotation.Bean

import org.springframework.jms.config.JmsListenerContainerFactory
import org.springframework.jms.config.SimpleJmsListenerContainerFactory

@SpringBootApplication
class DemoApplication {

  @Bean
  JmsListenerContainerFactory<SimpleJmsListenerContainerFactory> myJmsContainerFactory(ConnectionFactory connectionFactory) {
    SimpleJmsListenerContainerFactory factory = new SimpleJmsListenerContainerFactory()
    factory.setConnectionFactory(connectionFactory)
    return factory
  }

  static void main(String[] args) {
    SpringApplication.run DemoApplication, args
  }

}
```

That's it, now all components required have been already set. Here we are going to create a new `Person` message and send it to message service.

```groovy
package com.jos.dem.springboot.jms.controller

import org.springframework.web.bind.annotation.RestController
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.beans.factory.annotation.Autowired

import com.jos.dem.springboot.jms.command.Command
import com.jos.dem.springboot.jms.command.PersonCommand
import com.jos.dem.springboot.jms.service.MessageService

@RestController
class DemoController {

  @Autowired
  MessageService messageService

  @RequestMapping('/')
  String index(){
    Command person = new PersonCommand(nickname:'josdem', email:'joseluis.delacruz@gmail.com')
    messageService.sendMessage(person)
    'Java Message Service'
  }
  
}
```

This is the output from our project execution:

```bash
MessageServiceImpl : Sending message
MessageListener    : Message Received <com.jos.dem.springboot.jms.command.PersonCommand@3a85a88c nickname=josdem email=joseluis.delacruz@gmail.com>
```

To run the project

```bash
gradle bootRun
```

To download the project

```bash
git clone https://github.com/josdem/spring-boot-jms.git
```

[Return to the main article](/techtalk/spring)



