+++
date = "2018-01-27T15:45:00-06:00"
tags = ["josdem", "techtalks","programming","technology","Spring Boot RESTful"]
categories = ["techtalk", "code"]
title = "Spring Boot RESTFul"
+++

The main idea behind creating a RESTfull application is that you will create logical resources (GET, POST, UPDATE and DELETE) consumed by clients usign HTTP request.

This resources should be nouns no verbs. In this example we will create a RESTful application using a person resource.

So we can have:

* GET /persons - Retrieves a list of persons
* GET /persons/1234 - Retrieves a specific person
* POST /persons - Creates a new person
* PUT /persons/1234 - Updates #1234 person
* DELETE /tickets/1234 - Delete #1234 person


## Setup
In order to do that you need to install [SDKMAN](http://sdkman.io/) if you are using Linux or Mac, or [posh-gvm](https://github.com/flofreud/posh-gvm) if you are using Windows. After that, you can easily install:

* Spring Boot
* Groovy
* Gradle

Then execute this command in your terminal.

```bash
spring init --dependencies=web --language=groovy --build=gradle spring-boot-restful
```

This is the `build.gradle file` generated:

```groovy
buildscript {
	ext {
		springBootVersion = '1.5.9.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'groovy'
apply plugin: 'org.springframework.boot'

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
	mavenCentral()
}

dependencies {
	compile('org.springframework.boot:spring-boot-starter-web')
	compile('org.codehaus.groovy:groovy')
	testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

To Download the Project:

```bash
git clone https://github.com/josdem/spring-boot-restful.git
```

To Run the project:

```bash
gradle bootRun
```

[Return to the main article](/techtalk/spring)
