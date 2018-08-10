+++
title =  "From Anonymous to Lambda"
description = "From Anonymous Class to Lambda Expression"
date = "2018-08-09T17:35:09-04:00"
tags = ["josdem", "techtalks","programming","technology","Java", "Lambda Expression"]
categories = ["techtalk", "code", "java"]
+++

Anonymous classes traditionally enable you to declare and instantiate a class at the same time. They are like local classes except that they do not have a name. With lambda expressions introduced in Java 8 we can convert anonymous classes to lambda expressions and with that we will have less code, clean code and more assertive functionality definition. This time I will show you how to transfrom a anonymous class to lambda expression in two steps.

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

```bash
git clone https://github.com/josdem/junit5-workshop.git
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

