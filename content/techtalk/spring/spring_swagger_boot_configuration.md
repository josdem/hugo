+++
title = "Spring Boot Swagger"
categories = ["techtalk", "code","spring boot"]
tags = ["josdem", "techtalks", "programming", "technology", "swagger", "springfox", "swagger in spring", "swagger spring boot configuration","spring boot"]
date = "2016-07-17T14:56:13-05:00"
description = "Swagger is a simple yet powerful representation of your RESTful API. With Swagger you can keep your documentation attached with the evolution of your code and with Swagger UI you'll have a web interface that allows you to easily create GET and POST request to your API."
+++

Swagger is a simple yet powerful representation of your RESTful API. With Swagger you can keep your documentation attached with the evolution of your code and with Swagger UI you'll have a web interface that allows you to easily create GET and POST request to your API. To know more about Swagger please visit it's official web site [here](https://swagger.io/).

**NOTE:** If you need to know what tools you need to have installed in yout computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot)

Then execute this command in your terminal:

```groovy
spring init --dependencies=web,lombok --language=java --build=gradle boot-configuration
```

This is the `build.gradle` generated file:


```groovy
buildscript {
	ext {
    springBootVersion = '1.5.12.RELEASE'
	}
	repositories {
		mavenCentral()
	}
	dependencies {
		classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
	}
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'

repositories {
  mavenCentral()
}

dependencies {
  compile 'org.springframework.boot:spring-boot-starter-web'
  compile 'org.projectlombok:lombok'
  testCompile 'org.springframework.boot:spring-boot-starter-test'
}
```

Then add swagger dependencies in your `build.gradle` file

```groovy
ext {
  springfoxVersion = '2.8.0'
}

dependencies {
  compile "io.springfox:springfox-swagger2:${springfoxVersion}"
  compile "io.springfox:springfox-swagger-ui:${springfoxVersion}"
}
```

Next step is to create a Swagger Configuration file:

```java
package com.jos.dem.swagger.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration
@EnableSwagger2
public class SwaggerConfig {

  @Bean
  public Docket api() {
    return new Docket(DocumentationType.SWAGGER_2)
    .select()
    .apis(RequestHandlerSelectors.any())
    .paths(PathSelectors.any())
    .build();
  }

}
```

Swagger 2 is enabled through the `@EnableSwagger2` annotation. After the [Docket](http://springfox.github.io/springfox/javadoc/2.7.0/index.html?springfox/documentation/spring/web/plugins/Docket.html) bean is defined, its `select()` method returns an instance of `ApiSelectorBuilder`, which provides a way to control the endpoints exposed by Swagger.

Predicates for selection of `RequestHandlers` can be configured with the help of `RequestHandlerSelectors` and `PathSelectors`. Using `any()` for both will make documentation for your entire API available through Swagger.

## Controller Documentation

Now it is time to see the most common annotations in a `RestController` in order to create our API documentation.

```java
package com.jos.dem.swagger.controller;

import static org.springframework.web.bind.annotation.RequestMethod.GET;
import static org.springframework.web.bind.annotation.RequestMethod.POST;

import java.util.List;

import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiImplicitParam;
import io.swagger.annotations.ApiImplicitParams;

import com.jos.dem.swagger.service.UserService;
import com.jos.dem.swagger.command.UserCommand;
import com.jos.dem.swagger.model.User;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Api(description="knows how receive manage user requests")
@RestController
@RequestMapping("/users/*")
public class UserController {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Autowired
  private UserService userService;

  @RequestMapping(method = GET)
  public List<User> getAll(){
    log.info("Getting all users");
    return userService.getAll();
  }

  @ApiImplicitParam(name = "uuid", value = "User's uuid", required = true, dataType = "string", paramType = "path")
  @RequestMapping(method = GET, value = "{uuid}")
  public User getByUuid(@PathVariable String uuid){
    log.info("Getting user by uuid: " + uuid);
    return userService.getByUuid(uuid);
  }

  @RequestMapping(method = POST, consumes="application/json")
  public User create(@RequestBody UserCommand command){
    log.info("Saving user: w/uuid: " + command.getUuid());
    return userService.create(command);
  }

}
```

* `@ApiImplicitParams` Represents a single parameter in an API Operation, in this case user's uuid.
* `@Api` is used to declare a Swagger resource API. Only classes that are annotated with @Api will be scanned by Swagger. You can also document your model using `@ApiModel`

```java
package com.jos.dem.swagger.command;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

@ApiModel(value="UserModel", description="Model who represents an user entity")
public class UserCommand {
  @ApiModelProperty(value = "User's uuid", allowableValues = "aphanumeric")
  private String uuid;
  @ApiModelProperty(value = "User's name", allowableValues = "text")
  private String name;
  @ApiModelProperty(value = "User's email", allowableValues = "email@domain")
  private String email;

  public void setUuid(String uuid){
    this.uuid = uuid;
  }

  public String getUuid(){
    return this.uuid;
  }

  public void setName(String name){
    this.name = name;
  }

  public String getName(){
    return this.name;
  }

  public void setEmail(String email){
    this.email = email;
  }

  public String getEmail(){
    return this.email;
  }

}
```

`@ApiModel` Provides additional information about Swagger models. This translates to the [Model Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/1.2.md#527-model-object) in the Swagger Specification.

The endpoint to create request to your API using Swagger is: [http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)

## Swagger Results

**View**

<img src="/img/techtalks/spring/swagger1.png">

**Get all users**

<img src="/img/techtalks/spring/swagger2.png">

**Get user by uuid**

<img src="/img/techtalks/spring/swagger3.png">

To download the project:

```bash
git clone https://github.com/josdem/swagger-spring.git
cd boot-configuration
```

To run the project:

```
gradle bootRun
```

[Return to the main article](/techtalk/spring)
