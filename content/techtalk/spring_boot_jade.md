+++
date = "2015-08-29T20:18:08-05:00"
draft = true
title = "Spring Boot with Jade"

+++
### Setup
We are going to create a basic project with Spring Boot and Jade. First we need to go to this site: [Spring Initializr](http://start.spring.io/) and create a new project with next elements:

* Type: Gradle Project
* Java version: 1.8
* Language: Groovy
* Packaging: Jar
* Dependencies:
  * Web
  * JPA
  * H2
  * Actuator

Next download your project, and unzip it in some directory that you want. Browse that directory and you should see the following structure:

```
.
│── build.gradle
└── src
   ├── main
   │   ├── groovy
   │   │   └── jade
   │   │       └── JadeApplication.groovy
   │   └── resources
   │       ├── application.properties
   │       ├── static
   │       └── templates
   └── test
       └── groovy
           └── jade
               └── JadeApplicationTests.groovy
```

This is the build.gradle file generated:

```
buildscript {
  ext {
    springBootVersion = '1.3.0.M2'
  }
  repositories {
    mavenCentral()
    maven { url "https://repo.spring.io/snapshot" }
    maven { url "https://repo.spring.io/milestone" }
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  }
}

apply plugin: 'groovy'
apply plugin: 'eclipse'
apply plugin: 'idea'
apply plugin: 'spring-boot'

jar {
  baseName = 'spring'
  version = '0.0.1-SNAPSHOT'
}
sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
  mavenCentral()
  maven { url "https://repo.spring.io/snapshot" }
  maven { url "https://repo.spring.io/milestone" }
}


dependencies {
  compile("org.springframework.boot:spring-boot-starter-actuator")
  compile("org.springframework.boot:spring-boot-starter-data-jpa")
  compile("org.springframework.boot:spring-boot-starter-web")
  compile("org.codehaus.groovy:groovy")
  runtime("com.h2database:h2")
  testCompile("org.springframework.boot:spring-boot-starter-test")
}

eclipse {
  classpath {
    containers.remove('org.eclipse.jdt.launching.JRE_CONTAINER')
    containers 'org.eclipse.jdt.launching.JRE_CONTAINER/org.eclipse.jdt.internal.debug.ui.launcher.StandardVMType/JavaSE-1.8'
    }
}

task wrapper(type: Wrapper) {
  gradleVersion = '2.3'
}
```

As you can see we have Spring boot 1.3.0.M2, with Groovy plugin, and dependencies such as actuator, data JPA, web, H2 database and boot starter testing framework.

Type in your command line:

```
gradle build
```

Go to the project home and you should see the following path there build/libs, run the jar file as follow:

```
java -jar spring-boot-jade-0.0.1-SNAPSHOT.jar
```

At this point you have an application Spring Boot running in 8080 port.

### Web flow setup
Now it is time to create some controller to receive requests. Let’s create a simple Controller in your package directory in my case is in $HOME_PROJECT/src/main/groovy/jade

```java
package jade

import org.springframework.stereotype.Controller
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.ResponseBody

@Controller
class HomeController {

  @RequestMapping('/')
  @ResponseBody
  def index() {
    println 'Hello World!'
  }

}
```

In your project home, type:

```
gradle run
```

Now you are able to browse [localhost](http://localhost:8080/) and see some response from your HomeController in console

```
Hello World!
```

### Jade integration
Jade is a template language implemented with JavaScript. Jade is a clean, whitespace sensitive syntax for writing html, for more information go to: Jade To integrate [Jade](http://jade-lang.com/) add this line to the build.gradle file:

```
compile("com.domingosuarez.boot:spring-boot-starter-jade4j:0.3.0")
```

Next create this file: $HOME_PROJECT/src/main/resources/templates/index.jade

```html
html
  head
    title Simple Jade
  body
    h1 Home page
    p Current time: #{currentTime}
```

Finally modify HomeController.groovy to render a Date() response as follow:

```java
package jade

import org.springframework.stereotype.Controller
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.ResponseBody
import org.springframework.ui.Model

@Controller
class HomeController {

  @RequestMapping('/')
  String index(Model model) {
    model.addAttribute 'currentTime', new Date()
    'index'
  }

}
```

Return to the main article. [Spring Boot](/techtalk/techtalks)


