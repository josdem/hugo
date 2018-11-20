+++
title =  "Spring Boot Profiles"
description = "Spring_boot_profiles"
date = "2018-11-19T19:33:49-05:00"
tags = ["josdem", "techtalks","programming","technology","spring boot profiles"]
categories = ["techtalk", "code", "spring boot profiles"]
+++

In this technical post, we'll discuss how to manage different properties files when we run a Spring Boot application, this is helpful when you need to manage several environments. **NOTE:** If you need to know what tools you need to have installed in your computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot)

Then execute this command in your terminal.

```bash
spring init --dependencies=webflux --build=gradle --language=java spring-boot-profiles
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

group = 'com.jos.dem.springboot.profiles'
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


bootRun {
  systemProperties = System.properties
}
```

With `systemProperties = System.properties` in the `bootRun` task we are specifying to read command line properties. Now let's add as many properties files as environmets we want, so in that way we can define which one will be active from command line. In this example we are going to define only DEV and QA.

`application-dev.properties`

```properties
user.url=https://dev.josdem.io/
```

`application-qa.properties`

```properties
user.url=https://qa.josdem.io/
```

Next, let's read `user.url` property using `@Value` annotation and print it using `CommandLineRunner` which is a call back interface in Spring Boot, when Spring Boot starts will call it and pass in args through a `run()` internal method.

```java
package com.jos.dem.springboot.profiles;

import org.springframework.context.annotation.Bean;
import org.springframework.boot.CommandLineRunner;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ProfilesApplication {

  @Value("${user.url}")
  private String userUrl;

  public static void main(String[] args) {
    SpringApplication.run(ProfilesApplication.class, args);
  }

  @Bean
  CommandLineRunner run(){
    return args -> {
      System.out.println("Environment: " + userUrl);
    };
  }

}
```

That's it, if you run this application you will see `user.url` property value selected.

```bash
gradle -Dspring.profiles.active=dev bootRun
```

**Using Maven**

You can do the same using Maven, the only difference is that you need to specify `--build=maven` parameter in the `spring init` command line:

```bash
spring init --dependencies=webflux --build=maven --language=java spring-boot-profiles
```

This is the `pom.xml` file generated:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem.springboot</groupId>
  <artifactId>profiles</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-boot-profiles</name>
  <description>This project shows how to use profiles in a Spring Boot project with Gradle and Maven</description>

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

To run the project using Maven

```bash
mvn -Dspring.profiles.active=dev spring-boot:run
```

To browse the code go [here](https://github.com/josdem/spring-boot-profiles), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-profiles.git
```

[Return to the main article](/techtalk/spring#Spring_Boot)



