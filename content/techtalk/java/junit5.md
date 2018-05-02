+++
title =  "Junit 5"
description = "Junit5"
date = "2018-05-02T15:27:13-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code", "java"]
+++

Junit 5 is the next generation of Junit. JUnit 5 requires Java 8 since it was born as JUnit Lambda project, that project wanted to take full advantage of the new features from Java 8, especially lambda expressions.

For Gradle you need to create a `build.gradle`project with this dependencies:

```groovy
apply plugin: "java"
apply plugin: "application"

repositories {
  mavenCentral()
}

dependencies {
  testCompile 'org.junit.jupiter:junit-jupiter-api:5.1.1'
  testRuntime 'org.junit.jupiter:junit-jupiter-engine:5.1.1'
}
```

Dependencies shows how to configure support for JUnit Jupiter based tests, configuring a `testCompile` dependency on the JUnit Jupiter API and a `testRuntime` dependency on the JUnit Jupiter TestEngine.





To download the code:

```bash
git clone https://github.com/josdem/junit-workshop.git
```

To test the code:

```bash
gradle test
```

[Return to the main article](/techtalk/java)

