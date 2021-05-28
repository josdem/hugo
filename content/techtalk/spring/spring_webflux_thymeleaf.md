+++
title = "Spring Webflux with Thymeleaf"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology","spring boot","Spring Webflux", "Thymeleaf"]
date = "2017-08-29T20:18:08-05:00"
description = "Basic project with Spring Webflux and Thymeleaf"
+++

If you need to render HTML for web and stand alone applications and want to have reactive Spring Webflux as backend, this technical post is for you because this time we will go through the process to create a basic project in Spring Webflux with Thymeleaf. **NOTE:** If you need to know what tools you need to have installed in your computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring/spring_boot)


Then execute this command in your terminal.

```bash
spring init --dependencies=webflux,lombok,thymeleaf --build=gradle --language=java spring-boot-thymeleaf
```

This is the build.gradle file generated:

```groovy
plugins {
	id 'org.springframework.boot' version '2.5.0'
	id 'io.spring.dependency-management' version '1.0.11.RELEASE'
	id 'java'
}

group = 'com.jos.dem.springboot.thymeleaf'
version = '1.0.0-SNAPSHOT'
sourceCompatibility = '15'

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-webflux'
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testImplementation 'io.projectreactor:reactor-test'
}

test {
	useJUnitPlatform()
}
```

Now, let's create a `RestController`

```java
package com.jos.dem.springboot.thymeleaf.controller;

import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;

@RestController
public class DemoController{

  @RequestMapping("/")
  public String index(){
    return "Hello World!";
  }

}
```

At this point you have an application Spring Boot running in 8080 port. Execute this command in order execute this project, to see the Hello World message you might need to open URL provided by Spring in a browser:

```bash
gradle bootRun
```

## Changing to Web Page

Next step is change from `RestController` to a `Controller` so we can render the Hello World message to a web page:

```groovy
package com.jos.dem.springboot.thymeleaf.controller;

import com.jos.dem.springboot.thymeleaf.model.Person;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class DemoController{

  @RequestMapping("/")
  public String index(final Model model){
    Person person = new Person("josdem", "joseluis.delacruz@gmail.com");
    model.addAttribute("person", person);
    return "index";
  }

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

[Thymeleaf](http://www.thymeleaf.org/) is a modern template engine for both web and standalone environments and Lombok is a great tool to avoid boilerplate code, for knowing more please go [here](https://projectlombok.org/). Let’s create a simple model so we can send it to a web page

```java
package com.jos.dem.springboot.thymeleaf.model;

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.Setter;

@AllArgsConstructor
@Getter
@Setter
public class Person{
  private String nickname;
  private String email;
}
```

Another change we must do in the `Controller` is to return model and view:

```java
package com.jos.dem.springboot.thymeleaf.controller;

import com.jos.dem.springboot.thymeleaf.model.Person;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
public class DemoController{

  @RequestMapping("/")
  public String index(final Model model){
    Person person = new Person("josdem", "joseluis.delacruz@gmail.com");
    model.addAttribute("person", person);
    return "index";
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

To browse the project go [here](https://github.com/josdem/spring-boot-thymeleaf), to Download the Project:

```bash
git clone https://github.com/josdem/spring-boot-thymeleaf.git
```

[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
