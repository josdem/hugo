+++
title =  "Spring Webflux Multi-Module"
description = "Wow to build a multi-module project using Gradle and Maven with Spring Webflux"
date = "2018-12-16T10:48:18-05:00"
tags = ["josdem", "techtalks","programming","technology","spring boot"]
categories = ["techtalk", "code", "spring boot"]
+++

In this technical post, we will review how to build a multi-module project using Gradle and Maven with Spring Boot. **NOTE:** If you need to know what tools you need to have installed in your computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot). Let's start creating a new Spring Boot project with Webflux as a library:

```bash
spring init --dependencies=webflux --build=gradle --language=java library
```

Here is the complete `build.gradle` file with jar and dependencyManagement closures included:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.4.RELEASE' apply false
  id 'io.spring.dependency-management' version '1.0.6.RELEASE'
  id 'java'
}

apply plugin: 'java'
apply plugin: 'io.spring.dependency-management'

jar {
  baseName = 'spring-boot-module-library'
  version = '0.0.1-SNAPSHOT'
}

group = 'com.jos.dem.springboot.module.library'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.11

repositories {
  mavenCentral()
}


dependencies {
  implementation('org.springframework.boot:spring-boot-starter-webflux')
  testImplementation('org.springframework.boot:spring-boot-starter-test')
  testImplementation('io.projectreactor:reactor-test')
}

dependencyManagement {
  imports{
    mavenBom org.springframework.boot.gradle.plugin.SpringBootPlugin.BOM_COORDINATES
  }
}
```

Since we are creating a library here, we want Spring Boot’s dependency management to be used in this project without applying Spring Boot’s plugin. The `SpringBootPlugin` class provides a `BOM_COORDINATES`, a downside of this method is that it forces us to specify the version of the dependency management plugin. Let's create a service so we can use it in our application module.

```java
package com.jos.dem.springboot.module.library.service;

import reactor.core.publisher.Mono;

import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Value;

@Service
public class MessageService {

  @Value("${message}")
  private String message;

  public Mono<String> getMessage(){
    return Mono.just(message);
  }

}
```

Next, let's create an application project, from the `$PROJECT_HOME:`

```bash
spring init --dependencies=webflux --build=gradle --language=java application
```

This is the `build.gradle` file generated:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.4.RELEASE'
  id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.module'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.11

repositories {
  mavenCentral()
}

dependencies {
  implementation('org.springframework.boot:spring-boot-starter-webflux')
  implementation project(':library')
  testImplementation('org.springframework.boot:spring-boot-starter-test')
  testImplementation('io.projectreactor:reactor-test')
}
```

That's it, we are importing our library as project dependency using Gradle implementation. Now it is time to create a controller and Spring Boot Application files.

```java
package com.jos.dem.springboot.module.controller;

import reactor.core.publisher.Mono;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.springboot.module.library.service.MessageService;

@RestController
public class ModuleController {

  @Autowired
  private MessageService messageService;

  @GetMapping("/")
  public Mono<String> index(){
    return messageService.getMessage();
  }

}
```

Spring Boot application:

```java
package com.jos.dem.springboot.module;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ModuleApplication {

  public static void main(String[] args) {
    SpringApplication.run(ModuleApplication.class, args);
  }

}
```

We need to provide a message for the service in the library via `$PROJECT_HOME/application/src/main/resources/application.properties`.

```properties
message=Hello World!
```

Finally add a `settings.gradle` to the root directory to drive the build at the top level.

```bash
rootProject.name = 'spring-boot-module'

include 'library'
include 'application'
```

To run the project: Build the library first and then run the application. From `$PROJECT_HOME`:

```bash
gradle build :application:bootRun
```

**Using Maven**

You can do the same using Maven, the only difference is that you need to specify `--build=maven` parameter in the `spring init` command line. This is the `library`
 pom file generated:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem.springboot.module</groupId>
  <artifactId>library</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.4.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>11</java.version>
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
  </dependencies>

</project>
```

Here is the application pom file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem.springboot.module</groupId>
  <artifactId>application</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.4.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>11</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
      <groupId>com.jos.dem.springboot.module</groupId>
      <artifactId>library</artifactId>
      <version>0.0.1-SNAPSHOT</version>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
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

And this is our `pom.xml` in the `$PROJECT_HOME` directory.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem.springboot.module</groupId>
  <artifactId>spring-boot-module</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>pom</packaging>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.4.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <modules>
    <module>library</module>
    <module>application</module>
  </modules>

</project>
```

Build the library first and then run the application. From `$PROJECT_HOME`:

```bash
mvn install spring-boot:run -pl application
```

To see the results go to this URL in your browser:

```bash
 http://localhost:8080/
```

To browse the project go [here](https://github.com/josdem/spring-boot-modules), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-modules.git
```

[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
