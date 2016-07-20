+++
categories = ["techtalk", "code"]
date = "2016-07-17T14:56:13-05:00"
tags = ["josdem", "techtalks", "programming", "technology", "swagger", "springfox", "swagger in spring", "swagger spring boot configuration"]
title = "Swagger in Spring"

+++

Swagger is a simple yet powerful representation of your RESTful API. With Swagger you can keep your documentation attached with the evolution of your code and with Swagger UI you'll have a web interface that allows you to easily create GET and POST request to your API.

## Spring Boot Configuration

First you need to add swagger dependencies in your `build.gradle` file

```
ext {
  springfoxVersion = '2.4.0'
}

dependencies {
  compile "io.springfox:springfox-swagger2:${springfoxVersion}"
  compile "io.springfox:springfox-swagger-ui:${springfoxVersion}"
}
```

The complete `build.gradle` looks like this:

```groovy
buildscript {
  ext {
    springBootVersion = '1.3.6.RELEASE'
  }
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  }
}

ext {
  springfoxVersion = '2.5.0'
}

apply plugin: 'groovy'
apply plugin: 'spring-boot'

jar {
  baseName = 'boot-configuration'
  version = '0.0.1-SNAPSHOT'
}

sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
  mavenCentral()
}

dependencies {
  compile 'org.springframework.boot:spring-boot-starter-web'
  compile 'org.codehaus.groovy:groovy'
  compile "io.springfox:springfox-swagger2:${springfoxVersion}"
  compile "io.springfox:springfox-swagger-ui:${springfoxVersion}"
  testCompile 'org.springframework.boot:spring-boot-starter-test'
}
```

Then you need to create a Swagger Configuration file:

```groovy
package com.jos.dem.swagger.config

import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import springfox.documentation.spring.web.plugins.Docket
import springfox.documentation.spi.DocumentationType
import springfox.documentation.builders.PathSelectors
import springfox.documentation.builders.RequestHandlerSelectors
import springfox.documentation.swagger2.annotations.EnableSwagger2

@Configuration
@EnableSwagger2
class SwaggerConfig{
  @Bean
  Docket api() {
    return new Docket(DocumentationType.SWAGGER_2)
    .select()
    .apis(RequestHandlerSelectors.any())
    .paths(PathSelectors.any())
    .build()
  }
}
```

## Controller Documentation

If you need to pass parameters to your controller using `GET`, you can document it as follow:

```groovy
package com.jos.dem.swagger.controller

import static org.springframework.web.bind.annotation.RequestMethod.GET

import org.springframework.web.bind.annotation.RestController
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RequestMethod
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.http.HttpStatus
import org.springframework.http.ResponseEntity
import io.swagger.annotations.Api
import io.swagger.annotations.ApiImplicitParam
import io.swagger.annotations.ApiImplicitParams

import com.jos.dem.swagger.service.UserService
import com.jos.dem.swagger.command.UserCommand
import com.jos.dem.swagger.model.User

@Api(description="knows how receive manage user requests")
@RestController
@RequestMapping("/users/*")
class UserController {

  @Autowired
  UserService userService

  @ApiImplicitParams([
  @ApiImplicitParam(name = "uuid", value = "User's uuid", required = true, dataType = "string", paramType = "query")
  ])
  @RequestMapping(method = GET, value = "/")
  User getUserByUuid(UserCommand command){
    userService.getByUuid(command.uuid)
  }
}
```

`@ApiImplicitParams` Represents a single parameter in an API Operation, in this case user's uuid. You can also document your model using `@ApiModel`

```groovy
package com.jos.dem.swagger.command

import io.swagger.annotations.ApiModel
import io.swagger.annotations.ApiModelProperty

@ApiModel(value="UserCommand", description="Generic command from users")
class UserCommand {
  @ApiModelProperty(value = "User's uuid", allowableValues = "aphanumeric")
  String uuid
  @ApiModelProperty(value = "User's name", allowableValues = "text")
  String name
  @ApiModelProperty(value = "User's email", allowableValues = "email@domain")
  String email
}
```

So when you do a POST Swagger uses the `@ApiModelProperty` to represent the data. Here is the controller with both methods.

```groovy

dckage com.jos.dem.swagger.controller

import static org.springframework.web.bind.annotation.RequestMethod.GET
import static org.springframework.web.bind.annotation.RequestMethod.POST

import org.springframework.web.bind.annotation.RestController
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RequestMethod
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.http.HttpStatus
import org.springframework.http.ResponseEntity
import io.swagger.annotations.Api
import io.swagger.annotations.ApiImplicitParam
import io.swagger.annotations.ApiImplicitParams

import com.jos.dem.swagger.service.UserService
import com.jos.dem.swagger.command.UserCommand
import com.jos.dem.swagger.model.User

@Api(description="knows how receive manage user requests")
@RestController
@RequestMapping("/users/*")
class UserController {

  @Autowired
  UserService userService

  @ApiImplicitParams([
  @ApiImplicitParam(name = "uuid", value = "User's uuid", required = true, dataType = "string", paramType = "query")
  ])
  @RequestMapping(method = GET, value = "/")
  User getUserByUuid(UserCommand command){
    userService.getByUuid(command.uuid)
  }

  @RequestMapping(method = POST, consumes="application/json")
  User create(@RequestBody UserCommand command){
    userService.create(command)
  }

}
```

## Swagger Results

**View**

<img src="/img/techtalks/spring/swagger1.png">

**Post**

<img src="/img/techtalks/spring/swagger2.png">

**Get**

<img src="/img/techtalks/spring/swagger3.png">

To download the project:

```bash
git clone git@github.com:josdem/swagger-spring-mvc.git
git fetch
git checkout feature/swagger-boot-conf
```

To run the project.

```
gradle jettyRun
```

[Return to the main article](/techtalk/spring)
