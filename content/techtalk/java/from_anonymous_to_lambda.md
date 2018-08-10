+++
title =  "From Anonymous to Lambda"
description = "From Anonymous Class to Lambda Expression"
date = "2018-08-09T17:35:09-04:00"
tags = ["josdem", "techtalks","programming","technology","Java", "Lambda Expression"]
categories = ["techtalk", "code", "java"]
+++

Anonymous classes traditionally enable you to declare and instantiate a class at the same time. They are like local classes except that they do not have a name. With lambda expressions introduced in Java 8 we can move anonymous classes to lambda expressions and with that we will have less code, clean code and more assertive functionality definition. This time I will show you how to transfrom a anonymous class to lambda expression in two steps.

First let's create a simple java project using Gradle, in order to do that, in your root project create a `build.gradle` file with this structure:

```groovy
apply plugin: "java"

repositories {
  mavenCentral()
}

dependencies {
  testCompile 'org.junit.jupiter:junit-jupiter-api:5.2.0'
  testRuntime 'org.junit.jupiter:junit-jupiter-engine:5.2.0'
}

test {
  useJUnitPlatform()

  reports {
    html.enabled = true
  }
}
```

We are going to use [JUnit 5](/techtalk/java/junit5/) in order to create test and validate we are building the right functionality. Now it is time to create a [Functional Interface](http://josdem.io/techtalk/java/functional_interfaces/), in Java is any with `@FunctionalInterface` annotation and with SAM(Single Abstract Method). It was introduced to facilitate [Lambda expressions](http://josdem.io/techtalk/java/lambda_expressions/).

```java
package com.jos.dem.anonymous;

@FunctionalInterface
public interface Greeting {

  String hello();

}
```

Anonymous classes are unnamed instances of an interface or base class that are defined and instantiated with a single new command.

```java
@Test
@DisplayName("Should show get message as anonymous class")
void shouldGetMessageWithAnonymous() {
  final String message = greetingService.call(new Greeting() {

    @Override
    public String hello() {
      return "Hello World!";
    }
  });

  assertEquals("Hello World!", message);
}
```

Here is our GreetingService class:

```java
package com.jos.dem.anonymous;

public interface GreetingService {
  String call(final Greeting greeting);
}
```

And a `HelloWold` greeting service implementation:

```java
package com.jos.dem.anonymous;

public class HelloWorld implements GreetingService {

  @Override
  public String call(Greeting greeting){
    return greeting.hello();
  }

}
```

So the first step to change anonymous class to lambda expression is to get rid of method name:

```java
@Test
@DisplayName("Should remove method name")
void shouldGetMessageWithoutMethodName() {
  final String message = greetingService.call(() -> {
    return "Hello World!";
  });

  assertEquals("Hello World!", message);
}
```

Second step is to remove return statement

```java
@Test
@DisplayName("Should remove return statement")
void shouldGetMessageWithoutReturnStatement() {
  final String message = greetingService.call(() -> "Hello World!");
  assertEquals("Hello World!", message);
}
```

Here is the complete test case:

```java
package com.jos.dem.anonymous;

import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;

public class GreetingTest {

  private GreetingService greetingService = new HelloWorld();

  @Test
  @DisplayName("Should show get message as anonymous class")
  void shouldGetMessageWithAnonymous() {
    final String message = greetingService.call(new Greeting() {
      @Override
      public String hello() {
        return "Hello World!";
      }
    });

    assertEquals("Hello World!", message);
  }

  @Test
  @DisplayName("Should remove method name")
  void shouldGetMessageWithoutMethodName() {
    final String message = greetingService.call(() -> {
        return "Hello World!";
    });

    assertEquals("Hello World!", message);
  }

  @Test
  @DisplayName("Should remove return statement")
  void shouldGetMessageWithoutReturnStatement() {
    final String message = greetingService.call(() -> "Hello World!");
    assertEquals("Hello World!", message);
  }

}
```

If you want to run this project using Maven, add this file instead of `build.gradle`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem</groupId>
  <artifactId>junit5</artifactId>
  <version>1.0-SNAPSHOT</version>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>${maven.compiler.source}</maven.compiler.target>

    <junit.jupiter.version>5.2.0</junit.jupiter.version>
    <junit.platform.version>1.2.0</junit.platform.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-api</artifactId>
      <version>${junit.jupiter.version}</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-params</artifactId>
      <version>${junit.jupiter.version}</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-engine</artifactId>
      <version>${junit.jupiter.version}</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <!-- JUnit 5 requires Surefire version 2.22.0 or higher -->
      <plugin>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.22.0</version>
      </plugin>
    </plugins>
  </build>

</project>
```

To download the code:

```bash
git clone https://github.com/josdem/anonymous-to-lambda-workshop.git
```

To run the project using Gradle:

```bash
gradle test
```

To run the project using Maven:

```bash
mvn test
```


[Return to the main article](/techtalk/java)

