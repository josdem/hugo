+++
title = "Spring Boot"
date = "2015-08-29T20:12:17-05:00"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology","java validation","spring boot"]
description = "It is a Spring project aim to create easy web or stand-alone applications."
+++

## What is Spring Boot?
Open source Java framework used to create micro services but also to create easy web or stand-alone applications. It provides the next features:

* No XML configuration is required
* Netty (Starter Webflux) or Tomcat (Starter Web)
* Convention over configuration
* Easy setup
* Very lightweight
* Easy to test

This post shows you how to create a simple Spring Boot project with this features:

* Gradle Build
* Maven Build
* Basic configuration

You can create this project from command line like this:

**Using Gradle**

```bash
spring init --dependencies=web --build=gradle --language=java spring-boot-setup
```

That command will generate a Spring project structure under `spring-boot-setup` folder. In order to do that you need to install [SDKMAN](http://sdkman.io/) if you are using Linux or Mac, or [posh-gvm](https://github.com/flofreud/posh-gvm) if you are using Windows. After that, you can easily set up this software development kits:

* Spring Boot
* Gradle
* Maven
* Java

Here is a `build.gradle` generated example.

```groovy
plugins {
	id 'org.springframework.boot' version '2.3.3.RELEASE'
	id 'io.spring.dependency-management' version '1.0.10.RELEASE'
	id 'java'
}

group = 'com.jos.dem.springboot.setup'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '13'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testImplementation('org.springframework.boot:spring-boot-starter-test') {
		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
}

test {
	useJUnitPlatform()
}
```

`org.springframework.boot` makes it easy to stand up our Spring application with very little configuration, `io.spring.dependency.management` plugin provides Maven dependency managent. `boot:spring-boot-starter-test` is an starter for Spring Boot applications with libraries including Junit Jupiter and Mockito. This project will generate an `DemoApplication.java` file.

```groovy
package com.jos.dem.springboot.setup;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }

}
```
`@SpringApplication` annotation allows Spring Boot to scan recursively for beans inside this package also enables auto-configuration, attemping to guess and configure what you might need, how cool is that?. Next step, let's create a simple rest controller as follow:


```groovy
package com.jos.dem.springboot.setup;

import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;

@RestController
public class DemoController {

  @RequestMapping("/")
  public String index(){
    return "Hello World!";
  }

}
```

Now, in order to run it, use this command line.

```
gradle bootRun
```

Finally you can go to this address: [http://localhost:8080/](http://localhost:8080/) and you will see our hello word message.

**Using Maven**

You can do the same using Maven, the only difference is that you need to specify `--build=maven` parameter in the `spring init` command line:

```bash
spring init --dependencies=web --build=maven --language=java spring-boot-setup
```

And when you run your project use this command:

```bash
mvn spring-boot:run
```

This is the `pom.xml` file generated:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.3.3.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.jos.dem.springboot.setup</groupId>
	<artifactId>spring-boot-setup</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>demo</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>13</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
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

To browse the code go [here](https://github.com/josdem/spring-boot-setup), to download the code:

```bash
git clone git@github.com:josdem/spring-boot-setup.git
```


[Return to the main article](/techtalk/spring#Spring_Boot)
