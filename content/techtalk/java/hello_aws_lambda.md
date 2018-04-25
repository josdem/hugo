+++
tags = ["josdem","techtalks","programming","technology","java"]
categories = ["techtalk","code","java"]
date = "2017-01-12T18:20:33-06:00"
title = "AWS Hello Lambda"
description = "AWS Lambda is currently an option to implement Serverless architecture which is a name to refer to applications that run in ephemeral containers, such architectures remove the need for the traditional always on server and with that we save a lot of money in up time server consuming."
+++

AWS Lambda is currently an option to implement Serverless architecture which is a name to refer to applications that run in ephemeral containers, such architectures remove the need for the traditional 'always on' server and with that we save a lot of money in up time server consuming.

In this post I will show you how to create a hello world concept using AWS arquitecture. First we need to create a Java basic project this time using [lazybones](https://github.com/pledbrook/lazybones).

```bash
lazybones create java-basic hello-aws-lambda
```

Previous command will create this structure

```bash
<proj>
      |
      +- src
          |
          +- main
          |     |
          |     +- java
          |
          +- test
          |   |
          |   +- java
```

Edit your `build.gradle` file to create a make a Jar task and add lambda AWS dependencies.

```groovy
apply plugin: "java"
apply plugin: "application"

version = '0.0.1'

task buildJar(type: Jar) {
  manifest {
    attributes 'Implementation-Title': 'Hello AWS Lambda',
    'Implementation-Version': version,
    'Main-Class': 'example.Hello'
  }
  baseName = project.name + '-all'
  from { configurations.compile.collect { it.isDirectory() ? it : zipTree(it) } }
  with jar
}

repositories {
  mavenCentral()
}

dependencies {
  compile 'com.amazonaws:aws-lambda-java-core:1.1.0'
  compile 'com.amazonaws:aws-lambda-java-events:1.1.0'
}
```

Then define Java classes under `project-dir/src/main/java/example/`

**Hello.java**

```java
package example;

import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;

public class Hello implements RequestHandler<HelloRequest, HelloResponse> {

  @Override
  public HelloResponse handleRequest(HelloRequest input, Context context) {
    return new HelloResponse(input.getInput());
  }

}
```

**HelloRequest.java**

```java
package example;

public class HelloRequest {

  private String input;

  public HelloRequest(String input) {
    this.input = input;
  }

  public HelloRequest() {
  }

  public String getInput() {
    return input;
  }

  public void setInput(String input) {
    this.input = input;
  }

}
```

**HelloResponse.java**

```java
package example;

public class HelloResponse {

  private String hello;

  public HelloResponse(String hello) {
    this.hello = hello;
  }

  public HelloResponse() {
  }

  public String getHello() {
    return hello;
  }

  public void setHello(String hello) {
    this.hello = hello;
  }

}
```

Use the following gradle command to generate your standalone .jar deployment file

```bash
gradle buildJar
```

This will generate the .jar file with all dependencies under `build/libs`.

That's it, test your code uploading the jar file in to the [AWS Lambda console](https://eu-central-1.console.aws.amazon.com/lambda/home?region=eu-central-1#/functions/hello-world-in-java?tab=code)

Click Test button and change the JSON content with the following:

```json
{
  "input": "josdem"
}
```

To download the code:

```bash
git clone https://github.com/josdem/java-topics.git
cd hello-aws-lambda
```

[Return to the main article](/techtalk/java)

