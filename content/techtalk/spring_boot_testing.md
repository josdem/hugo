+++
date = "2015-12-02T22:18:18-06:00"
draft = true
title = "Spring Boot Testing"

+++

We can test our application using Junit or Spock, and we will see both.

Spring Boot provides a @SpringApplicationConfiguration annotation as an alternative to the standard @ContextConfiguration annotation.

```groovy
package com.jos.dem.jmailer

import static org.junit.Assert.assertEquals

import org.junit.Test
import org.junit.runner.RunWith
import org.springframework.boot.test.SpringApplicationConfiguration
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.mail.javamail.JavaMailSenderImpl

@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = JmailerApplication.class)
class JmailerApplicationTests {

  @Autowired
  JavaMailSenderImpl javaMailSender

  @Test
  void shouldLoadContext() {
    assertEquals('smtp.gmail.com', javaMailSender.host)
    assertEquals(587, javaMailSender.port)
  }

}
```

We can run this test with:

```bash
gradle -Dtest.single=JmailerApplicationTests test -Dspring.config.location=$HOME/.spring/application-DEVELOPMENT.yml
```

Using Spock we need to use @ContextConfiguration to load our context, here is an integration test example:

```groovy
package com.jos.dem.jmailer.service

import spock.lang.Specification

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.test.SpringApplicationContextLoader
import org.springframework.boot.test.IntegrationTest
import org.springframework.test.context.ContextConfiguration

import com.jos.dem.jmailer.JmailerApplication
import com.jos.dem.jmailer.command.MessageCommand

@ContextConfiguration(loader = SpringApplicationContextLoader.class, classes = JmailerApplication.class)
@IntegrationTest
class NotificationServiceSpec extends Specification {

  @Autowired
  NotificationService notificationService

  String email = 'joseluis.delacruz@gmail.com'

  void "should send an email"(){
    given:"A MessageCommand"
      def messageCommand = new MessageCommand(email:email, message:'Hello from spock')
    when:"We send notification"
      def result = notificationService.sendNotification(messageCommand)
    then:"We expect email sent"
      result
  }
}
```

For this to work you need to have `org.spockframework:spock-spring` on the dependencies.

```groovy
buildscript {
  ext {
    springBootVersion = '1.3.0.RELEASE'
  }
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  }
}

apply plugin: 'groovy'
apply plugin: 'spring-boot'

jar {
  baseName = 'jmailer-spring-boot'
  version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
  mavenCentral()
}

dependencies {
  compile 'org.springframework.boot:spring-boot-starter-web'
  compile 'org.springframework.boot:spring-boot-starter-aop'
  compile 'org.springframework.boot:spring-boot-starter-mail'
  compile 'org.springframework.boot:spring-boot-starter-thymeleaf'
  compile 'org.springframework:spring-jms'
  compile 'org.apache.activemq:activemq-broker'
  compile 'org.codehaus.groovy:groovy:2.4.5'
  compile 'com.google.code.gson:gson:2.3.1'
  compile 'org.freemarker:freemarker:2.3.23'
  testCompile 'org.spockframework:spock-spring:1.0-groovy-2.4'
  testCompile 'org.springframework.boot:spring-boot-starter-test'
}

task wrapper(type: Wrapper) {
  gradleVersion = '2.7'
}

bootRun {
  systemProperties = System.properties
}

test {
  systemProperties = System.properties
}
```

We can run all the test whit this command:

```bash
gradle test -Dspring.config.location=$HOME/.spring/application-DEVELOPMENT.yml
```

To download the project

```bash
git clone git@github.com:josdem/jmailer-spring-boot.git
git fetch
git checkout feature/testing
```

[Return to the main article](/techtalk/spring)