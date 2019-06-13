+++
title = "Spring Boot JMS"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology", "spring boot jms", "java message service", "spring boot"]
date = "2018-02-26T09:08:25-06:00"
description = "The Java Message Service is an API for sending messages between two or more clients. It is an implementation to Producer-Consumer Design Pattern"
+++

[Java Message Service](https://docs.spring.io/spring/docs/3.0.x/spring-framework-reference/html/jms.html) es una API para envio de mensajes entre uno o mas clientes. Es la implementaciión [Producer-Consumer Design Pattern](https://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem). El caso de uso es típicamente cuando tienes procesos que consumen tiempo de procesamiento y requieres que la respuesta sea manejada asíncronamente, así evitas que los clientes esperen por una respuesta. Para poner esto en contexto vamos a pensar en el escenario donde necesitas enviar un correo electrónico. Enviar un email consume tiempo así que es buena idea poner el mensaje en una cola de mensajes, de esta manera podemos continuar con el flujo de negocio sin forzar al cliente que espere hasta que el mensade de correo electrónico sea entregado. En este ejemplo te mostraré como usar JMS en una aplicación Spring Boot. **Nota:** Si quieres saber que herramientas necesitas tener instaladas en tu computadora para poder familiarizarte con Spring Boot por favor visita mi previo post: [Spring Boot](/techtalk/spring/spring_boot). Entonces ejecuta este comando desde la terminal.

```bash
spring init --dependencies=webflux,activemq,lombok --language=java --build=gradle spring-boot-jms
```

Aquí está el `build.gradle` generado:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.5.RELEASE'
  id 'java'
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.jms'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 11

repositories {
  mavenCentral()
}

dependencies {
  implementation('org.springframework.boot:spring-boot-starter-webflux')
  implementation("org.springframework.boot:spring-boot-starter-activemq")
  compileOnly('org.projectlombok:lombok')
  annotationProcessor('org.projectlombok:lombok')
  testImplementation('org.springframework.boot:spring-boot-starter-test')
}
```

Después agrega estas dependencias:

```groovy
implementation('org.apache.commons:commons-lang3')
implementation('org.apache.activemq:activemq-broker')
```

Ahora, primero vamos a crear un `MessageService` para poder entregar mensajes a la cola de espera.

```groovy
package com.jos.dem.springboot.jms.service

import com.jos.dem.springboot.jms.command.Command

interface MessageService {

  void sendMessage(final Command command)

}
```

Este es nuestra implementación del servicio `MessageServiceImpl`:

```groovy
package com.jos.dem.springboot.jms.service.impl;

import javax.jms.JMSException;
import javax.jms.ObjectMessage;
import javax.jms.Message;
import javax.jms.Session;

import org.springframework.stereotype.Service;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.jms.core.MessageCreator;
import org.springframework.jms.annotation.EnableJms;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.springboot.jms.command.Command;
import com.jos.dem.springboot.jms.service.MessageService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
@EnableJms
public class MessageServiceImpl implements MessageService {

  @Autowired
  private JmsTemplate jmsTemplate;

  private Logger log = LoggerFactory.getLogger(this.getClass());

  public void sendMessage(final Command command) {
    jmsTemplate.send("destination", (Session session) -> {
      ObjectMessage message = session.createObjectMessage();
      message.setObject(command);
      return message;
    });
  }

}
```

Donde:

* `@EnableJms` Descubre métodos anotados con `@JmsListener`.
* `JmsTemplate` Envía mensajes al destino JMS.
* `Command` Es un contrato para serializar los objetos POJO.

```groovy
package com.jos.dem.springboot.jms.command

import java.io.Serializable

interface Command extends Serializable {}
```

Este el el mensaje que vamos a enviar.

```groovy
package com.jos.dem.springboot.jms.command;

import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class PersonCommand implements Command {
	private String nickname;
	private String email;
}
```

Ahora que tenemos todas la entidades que necesitamos para enviar un mensaje, vamos a especificar las necesarias para procesar los mensajes.

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
  void receiveMessage(Message message) {
    Object command =  ((ObjectMessage) message).getObject()
    log.info "Message Received ${command.dump()}"
  }

}
```

Así como puedes ver `JmsTemplate` está enviando un mensaje a `destination` y `@JmsListener` está esperando por mensaje desde `destination`, otra parte imporante en el rompecabnezas es el contenerdo JMS llamado `myJmsContainerFactory` que está definido en nuestra aplicación Spring Boot como un bean.

```groovy
package com.jos.dem.springboot.jms;

import javax.jms.ConnectionFactory;

import org.springframework.context.annotation.Bean;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import org.springframework.jms.config.JmsListenerContainerFactory;
import org.springframework.jms.config.SimpleJmsListenerContainerFactory;

@SpringBootApplication
public class DemoApplication {

  @Bean
  public JmsListenerContainerFactory<?> myJmsContainerFactory(ConnectionFactory connectionFactory) {
    SimpleJmsListenerContainerFactory factory = new SimpleJmsListenerContainerFactory();
    factory.setConnectionFactory(connectionFactory);
    return factory;
  }

  public static void main(String[] args) {
	  SpringApplication.run(DemoApplication.class, args);
  }

}
```

That's it, now all components required have been already set. Here we are going to create a new `Person` message and send it to message service.

```groovy
package com.jos.dem.springboot.jms.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.springboot.jms.command.Command;
import com.jos.dem.springboot.jms.command.PersonCommand;
import com.jos.dem.springboot.jms.service.MessageService;

@RestController
public class DemoController {

  @Autowired
  private MessageService messageService;

  @GetMapping("/")
  public String index(){
  	Command person = new PersonCommand("josdem","joseluis.delacruz@gmail.com");
  	messageService.sendMessage(person);
  	return "Java Message Service";
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

This is the pom.xml file generated:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem.springboot</groupId>
  <artifactId>jms</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>demo</name>
  <description>This project shows how to use JMS (Java Message Service) in a Spring Boot project</description>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.5.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>11</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-activemq</artifactId>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
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
