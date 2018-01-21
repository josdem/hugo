+++
date = "2017-08-29T20:18:08-05:00"
draft = true
title = "Spring Boot with Thymeleaf"

+++
## Setup
This time I will show you how to create a basic project in Spring Boot with Thymeleaf, in order to do that you need to install [SDKMAN](http://sdkman.io/) if you are using Linux or Mac, or [posh-gvm](https://github.com/flofreud/posh-gvm) if you are using Windows. After that, you can easily install:

* Spring Boot
* Groovy
* Gradle

Then execute this command in your terminal.

```bash
spring init --dependencies=web --language=groovy --build=gradle spring-boot-thymeleaf
```

This is the build.gradle file generated:

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

Now, let's create a `RestController`

```groovy
package com.jos.dem.springboot.thymeleaf.controller

import org.springframework.web.bind.annotation.RestController
import org.springframework.web.bind.annotation.RequestMapping

@RestController
class DemoController{

  @RequestMapping("/")
  String index(){
    'Hello World!'
  }

}
```

At this point you have an application Spring Boot running in 8080 port. Execute this command in order to see the Hello World message:

```bash
gradle bootRun
```

## Changing to Web Page

Next step is change from `RestController` to a `Controller` so we can render the Hello World message to a web page:

```groovy
package com.jos.dem.springboot.thymeleaf.controller

import org.springframework.stereotype.Controller
import org.springframework.web.bind.annotation.RequestMapping

@Controller
class DemoController{

  @RequestMapping("/")
  String index(){
    'index'
  }

}
```

Next we need to add Thymeleaf dependency to the `build.gradle` file

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

version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
  mavenCentral()
}


dependencies {
  compile('org.springframework.boot:spring-boot-starter-web')
  compile('org.codehaus.groovy:groovy')
  compile('org.springframework.boot:spring-boot-starter-thymeleaf')
  testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

Finally we can create a simple web page in `$PROJECT_HOME/src/main/resources/templates`

```html
<html>
  Hello World!
</html>
```

If you run the application again, you should see the index web page showing the Hello World message in `localhost:8080`

## Sending Model Using Thymeleaf Template

[Thymeleaf](http://www.thymeleaf.org/) is a modern template engine for both web and standalone environments. Let’s create a simple model so we can send it to a web page

```groovy
package com.jos.dem.springboot.thymeleaf.model

class Person{
  String nickname
  String email
}
```

Another change we must do in the `Controller` is to return model and view:

```groovy
package com.jos.dem.springboot.thymeleaf.controller

import org.springframework.stereotype.Controller
import org.springframework.web.servlet.ModelAndView
import org.springframework.web.bind.annotation.RequestMapping

import com.jos.dem.springboot.thymeleaf.model.Person

@Controller
class DemoController{

  @RequestMapping("/")
  ModelAndView index(){
    Person person = new Person(nickname:'josdem',email:'joseluis.delacruz@gmail.com')
    ModelAndView modelAndView = new ModelAndView('index')
    modelAndView.addObject(person)
    modelAndView
  }

}
```

And you can see the model in the template using some Thymeleaf tags:

```html
<html>
  <h3 th:text="${person.nickname}" />
  <p th:text="${person.email}" />
</html>
```

To Download the Project:

```bash
git clone https://github.com/josdem/spring-boot-thymeleaf.git
```

[Return to the main article](/techtalk/spring)


