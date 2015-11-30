+++
date = "2015-11-29T21:10:13-06:00"
draft = true
title = "Jmailer Spring Boot"

+++

Let's create a project with web dependency from command line.

```bash
spring init --dependencies=web --build=gradle jmailer-spring-boot
```

That command will generate a Spring project structure under jmailer-spring-boot folder, I will show you a short version of build.gradle generated.

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
  compile 'org.codehaus.groovy:groovy:2.4.5'
  testCompile 'org.springframework.boot:spring-boot-starter-test'
}

task wrapper(type: Wrapper) {
  gradleVersion = '2.7'
}
```

This project will generate an DemoApplication.java file, rename it as JmailerApplication.groovy in a package structure you want.

```groovy
package com.jos.dem.jmailer

import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.SpringBootApplication

@SpringBootApplication
class JmailerApplication {

  static void main(String[] args) {
    SpringApplication.run JmailerApplication.class, args
  }

}
```

This is my project structure so far

```bash
.
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew
├── gradlew.bat
└── src
    ├── main
    │   ├── groovy
    │   │   └── com
    │   │       └── jos
    │   │           └── dem
    │   │               └── jmailer
    │   │                   ├── controller
    │   │                   │   └── SimpleController.groovy
    │   │                   └── JmailerApplication.groovy
    │   └── resources
    │       ├── application.properties
    │       └── static
    └── test
        └── groovy
            └── com
                └── jos
                    └── dem
                        └── jmailer
                            └── JmailerApplicationTests.groov
```

Next create a groovy file with simple rest controller description as follow:

```groovy
package com.jos.dem.jmailer.controller

import org.springframework.web.bind.annotation.RestController
import org.springframework.web.bind.annotation.RequestMapping

@RestController
class SimpleController {

  @RequestMapping("/")
  String index() {
    "Greetings from Spring Boot!"
  }

}
```

Finally run it from command line.

```bash
gradle bootRun
```

To download the project:

```bash
git clone git@github.com:josdem/jmailer-spring-boot.git
git fetch
git checkout setup
```

[Return to the main article](/techtalk/spring)
