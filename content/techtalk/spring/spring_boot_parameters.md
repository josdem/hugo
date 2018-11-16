+++
title =  "Spring Boot Parameters"
description = "In this technical post, we’ll discuss how to pass system properties as arguments to a Spring Boot application"
date = "2018-11-16T14:21:45-05:00"
tags = ["josdem", "techtalks","programming","technology","spring boot"]
categories = ["techtalk", "code","spring boot"]
+++

In this technical post, we’ll discuss how to pass system properties as arguments to a Spring Boot application. **NOTE:** If you need to know what tools you need to have installed in your computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot)

Then execute this command in your terminal.

```bash
spring init --dependencies=webflux --language=java --build=maven spring-boot-parameters
```

This is the `build.gradle file` generated:

```groovy
buildscript {
  ext {
    springBootVersion = '2.1.0.RELEASE'
  }
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  }
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.parameter'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
  mavenCentral()
}


dependencies {
	implementation('org.springframework.boot:spring-boot-starter-webflux')
	testImplementation('org.springframework.boot:spring-boot-starter-test')
	testImplementation('io.projectreactor:reactor-test')
}
```

Now, let's read `$user` system properties using `@Value` annotation and print it in command line, the `CommandLineRunner` is a call back interface in Spring Boot, when Spring Boot starts will call it and pass in args through a `run()` internal method.

```java
package com.jos.dem.springboot.parameter;

import org.springframework.context.annotation.Bean;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ParameterApplication {

  @Value("${user}")
  private String user;

  public static void main(String[] args) {
    SpringApplication.run(ParameterApplication.class, args);
  }

  @Bean
  CommandLineRunner run(){
    return args -> {
      System.out.println("user: " + user);
    };
  }

}
```

That's it, if you run this application you will see `$user` system properties value in your command line.

```bash
2018-11-16 14:17:46.984  INFO 27684 --- [main] c.j.d.s.parameter.ParameterApplication : Started ParameterApplication in 1.524 seconds (JVM running for 1.862)
user: moralej3
```

Also you are able to set custom system properties variables such as:

```bash
export nickname=josdem
echo $nickname
```

And read them in the same way.

**Using Maven**

You can do the same using Maven, the only difference is that you need to specify `--build=maven` parameter in the `spring init` command line:

```bash
spring init --dependencies=webflux --language=java --build=maven spring-boot-parameters
```

This is the `pom.xml` file generated:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem.springboot</groupId>
  <artifactId>parameter</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>demo</name>
  <description>Shows how to run a Spring Boot project with custom parameters</description>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.0.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-webflux</artifactId>
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

To run the project using Gradle

```bash
gradle bootRun
```

To run the project using Maven

```bash
mvn spring-boot:run
```

To browse the code go [here](https://github.com/josdem/spring-boot-parameters), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-parameters.git
```

[Return to the main article](/techtalk/spring#Spring_Boot)



