+++
title = "Spring Boot JMS"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology", "spring boot jms", "java message service", "spring boot"]
date = "2018-02-26T09:08:25-06:00"
description = "The Java Message Service is an API for sending messages between two or more clients. It is an implementation to Producer-Consumer Design Pattern"
+++

[Java Message Service](https://docs.spring.io/spring/docs/3.0.x/spring-framework-reference/html/jms.html) is an API for sending and receiving messages. It is an implementation to [Producer-Consumer Design Pattern](https://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem). This technique is usually implemented when you have a time consuming process and you need to avoid that a client is waiting for completing that process. To put this in context let’s think about a scenario where we could use it. The first thing that comes to my mind is an email delivery process. Sending an email consumes time and we can put email delivery as a message in a queue, so we can continue with our business flow without force to the client to wait until this email is deliver. In this example, we will see how to use JMS in a Spring Boot application. **NOTE:** If you need to know what tools you need to have installed in your computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring/spring_boot)

Then execute this command in your terminal.

```bash
spring init --dependencies=webflux,activemq,lombok --language=java --build=gradle spring-boot-jms
```

This is the `build.gradle file` generated:

```groovy
plugins {
  id 'org.springframework.boot' version '2.4.5'
  id 'io.spring.dependency-management' version '1.0.11.RELEASE'
  id 'java'
}

group = 'com.jos.dem.springboot.jms'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '15'

configurations {
  compileOnly {
    extendsFrom annotationProcessor
  }
}

repositories {
  mavenCentral()
}

dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-activemq'
  implementation 'org.springframework.boot:spring-boot-starter-webflux'
  implementation 'org.apache.commons:commons-lang3'
  implementation 'org.apache.activemq:activemq-broker'
  compileOnly 'org.projectlombok:lombok'
  annotationProcessor 'org.projectlombok:lombok'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
  testImplementation 'io.projectreactor:reactor-test'
}

test {
  useJUnitPlatform()
}
```

Next add this dependencies:

```groovy
implementation('org.apache.commons:commons-lang3')
implementation('org.apache.activemq:activemq-broker')
```

First, lets create a `MessageService` to deliver messages to the queue.

```java
package com.jos.dem.springboot.jms.service;

import com.jos.dem.springboot.jms.command.Command;

public interface MessageService {

  void sendMessage(final Command command);
}
```

This is our `MessageServiceImpl` implementation class:

```java
package com.jos.dem.springboot.jms.service.impl;

import com.jos.dem.springboot.jms.command.Command;
import com.jos.dem.springboot.jms.service.MessageService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.jms.annotation.EnableJms;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.stereotype.Service;

import javax.jms.ObjectMessage;
import javax.jms.Session;

@Slf4j
@Service
@EnableJms
@RequiredArgsConstructor
public class MessageServiceImpl implements MessageService {

  private final JmsTemplate jmsTemplate;

  public void sendMessage(final Command command) {
    jmsTemplate.send(
        "destination",
        (Session session) -> {
          ObjectMessage message = session.createObjectMessage();
          message.setObject(command);
          return message;
        });
  }
}
```

Where:

* `@EnableJms` Discovers methods annotated with `@JmsListener`.
* `JmsTemplate` Sends messages to a JMS destination
* `Command` Is a contract, so we can make serializable specific POJO.

```java
package com.jos.dem.springboot.jms.command

import java.io.Serializable

interface Command extends Serializable {}
```

And this is the message object we are going to send.

```java
package com.jos.dem.springboot.jms.command;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class PersonCommand implements Command {
  private String nickname;
  private String email;
}
```

Now we have all entities we need to set in order to send a message, next we need to specify the entities to receive and process messages

```java
package com.jos.dem.springboot.jms.messengine;

import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.builder.ToStringBuilder;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.ObjectMessage;

@Slf4j
@Component
public class MessageListener {

  @JmsListener(destination = "destination", containerFactory = "myJmsContainerFactory")
  public void receiveMessage(Message message) throws JMSException {
    Object command = ((ObjectMessage) message).getObject();
    log.info("Message Received: " + ToStringBuilder.reflectionToString(command));
  }
}
```

As you can see `JmsTemplate` is sending a message to the `destination` and `@JmsListener` is waiting for new messages from `destination`, the another important part in this puzzle is the JMS container called `myJmsContainerFactory` which is defined in our Spring Boot application class as a bean.

```java
package com.jos.dem.springboot.jms;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.jms.config.JmsListenerContainerFactory;
import org.springframework.jms.config.SimpleJmsListenerContainerFactory;

import javax.jms.ConnectionFactory;

@SpringBootApplication
public class JmsDemoApplication {

  @Bean
  public JmsListenerContainerFactory<?> myJmsContainerFactory(ConnectionFactory connectionFactory) {
    SimpleJmsListenerContainerFactory factory = new SimpleJmsListenerContainerFactory();
    factory.setConnectionFactory(connectionFactory);
    return factory;
  }

  public static void main(String[] args) {
    SpringApplication.run(JmsDemoApplication.class, args);
  }
}
```

That's it, now all components required have been already set. Here we are going to create a new `Person` message and send it to message service.

```java
package com.jos.dem.springboot.jms.controller;

import com.jos.dem.springboot.jms.command.Command;
import com.jos.dem.springboot.jms.command.PersonCommand;
import com.jos.dem.springboot.jms.service.MessageService;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Mono;

@Slf4j
@RestController
@RequiredArgsConstructor
public class JmsController {

  private final MessageService messageService;

  @GetMapping("/")
  public Mono<String> index() {
    log.info("Sending message");
    Command person = new PersonCommand("josdem", "joseluis.delacruz@gmail.com");
    messageService.sendMessage(person);
    return Mono.just("Java Message Service");
  }
}
```

Now if you start our Spring Boot Application:

```bash
gradle bootRun
```

And hit this endopoint from command line:

```bash
curl http://localhost:8080/
```

You should be able to get this output:

```bash
MessageServiceImpl : Sending message
MessageListener    : Message Received <com.jos.dem.springboot.jms.command.PersonCommand@3a85a88c nickname=josdem email=joseluis.delacruz@gmail.com>
```

**Using Maven**

You can do the same using Maven, the only difference is that you need to specify `--build=maven` parameter in the `spring init` command line:

```bash
spring init --dependencies=webflux,activemq,lombok --language=java --build=maven spring-boot-jms
```

This is the pom.xml file generated:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.4.5</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
  <groupId>com.jos.dem.springboot</groupId>
  <artifactId>spring-boot-jms</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>spring-boot-jms</name>
  <description>This project shows how to use JMS (Java Message Service) in a Spring Boot project</description>

  <properties>
    <java.version>15</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-activemq</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>

    <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
    </dependency>
    <dependency>
      <groupId>org.apache.activemq</groupId>
      <artifactId>activemq-broker</artifactId>
    </dependency>

    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>io.projectreactor</groupId>
      <artifactId>reactor-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <configuration>
          <excludes>
            <exclude>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
            </exclude>
          </excludes>
        </configuration>
      </plugin>
    </plugins>
  </build>

</project>
```

To run the project with Gradle:

```bash
gradle bootRun
```

To run the project with Maven:

```bash
mvn spring-boot:run
```

To browse the project go [here](https://github.com/josdem/spring-boot-jms), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-jms.git
```

[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
