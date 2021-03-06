+++
title = "Usando JMS en Spring Webflux"
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
  id 'org.springframework.boot' version '2.2.0.RELEASE'
  id 'io.spring.dependency-management' version '1.0.8.RELEASE'
  id 'java'
}

group = 'com.jos.dem.springboot.jms'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 11

configurations {
  compileOnly {
    extendsFrom annotationProcessor
  }
}

repositories {
  mavenCentral()
}

dependencies {
  implementation('org.springframework.boot:spring-boot-starter-webflux')
  implementation("org.springframework.boot:spring-boot-starter-activemq")
  compileOnly('org.projectlombok:lombok')
  annotationProcessor('org.projectlombok:lombok')
  testImplementation('org.springframework.boot:spring-boot-starter-test') {
    exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
  }
  testImplementation 'io.projectreactor:reactor-test'
}

test {
  useJUnitPlatform()
}
```

Después agrega estas dependencias:

```groovy
implementation('org.apache.commons:commons-lang3')
implementation('org.apache.activemq:activemq-broker')
```

Ahora, primero vamos a crear un `MessageService` para poder entregar mensajes a la cola de espera.

```java
package com.jos.dem.springboot.jms.service

import com.jos.dem.springboot.jms.command.Command

interface MessageService {

  void sendMessage(final Command command)

}
```

Este es nuestra implementación del servicio `MessageServiceImpl`:

```java
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

```java
package com.jos.dem.springboot.jms.command

import java.io.Serializable

interface Command extends Serializable {}
```

Este el el mensaje que vamos a enviar.

```java
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

```java
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

```java
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

Así es, ahora todos los componentes requeridos han sido definidos. Vamos creando una nueva entidad `Person` que sirva como mensaje para ser enviado por nuestro service.

```java
package com.jos.dem.springboot.jms.controller;

import reactor.core.publisher.Mono;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.springboot.jms.command.Command;
import com.jos.dem.springboot.jms.command.PersonCommand;
import com.jos.dem.springboot.jms.service.MessageService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@RestController
public class DemoController {

	@Autowired
  private MessageService messageService;

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @GetMapping("/")
  public Mono<String> index(){
    log.info("Sending message");
  	Command person = new PersonCommand("josdem","joseluis.delacruz@gmail.com");
  	messageService.sendMessage(person);
  	return Mono.just("Java Message Service");
  }

}
```

Ahora si iniciamos nuestra aplicación Spring Boot:

```bash
gradle bootRun
```

Y al consultar el endpoint desde la línea de comando:

```bash
curl http://localhost:8080/
```

Deberíamos poder ver esta salida:

```bash
MessageServiceImpl : Sending message
MessageListener    : Message Received <com.jos.dem.springboot.jms.command.PersonCommand@3a85a88c nickname=josdem email=joseluis.delacruz@gmail.com>
```

**Using Maven**

Tú puedes hacer lo mismo usando Maven, la única diferencia es que tienes que específicar el parámetro `--build=maven` en el comando `spring init`:

```bash
spring init --dependencies=webflux,activemq,lombok --language=java --build=maven spring-boot-jms
```

Este es el `pom.xml` generado:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.0.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <groupId>com.jos.dem.springboot</groupId>
  <artifactId>jms</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>
  <name>spring-boot-jms</name>
  <description>This project shows how to use JMS (Java Message Service) in a Spring Boot project</description>

  <properties>
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
`
```

Para correr el proyecto con Gradle:

```bash
gradle bootRun
```

Para correr el proyecto con Maven:

```bash
mvn spring-boot:run
```

Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/spring-boot-jms), para descargar el proyecto:

```bash
git clone git@github.com:josdem/spring-boot-jms.git
```

[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
