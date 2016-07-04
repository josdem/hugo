+++
date = "2015-12-06T15:43:14-06:00"
draft = true
title = "Spring Boot Externalization"

+++

In this example we are going to externalize our Jmailer application using yaml.

First we need to create our yml file in ${USER_HOME}/.spring/application-DEVELOPMENT.yml

```yaml
email:
  username: username@email.com
  password: mypassword
  template: message.ftl
  subject: Hello from Spring Boot!
```

Next we need to add bootRun system properties in our build.gradle file

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
  testCompile 'org.springframework.boot:spring-boot-starter-test'
}

task wrapper(type: Wrapper) {
  gradleVersion = '2.7'
}

bootRun {
  systemProperties = System.properties
}
```

In order to get our yaml file as system properties we need to execute this command:

```bash
gradle bootRun -Dspring.config.location=$HOME/.spring/application-DEVELOPMENT.yml
```

To download the project

```bash
git clone git@github.com:josdem/jmailer-spring-boot.git
git fetch
git checkout feature/mail
```

[Return to the main article](/techtalk/spring)
