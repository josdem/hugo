+++
title = "Spring Boot Oauth2 with Google"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology", "Spring Boot", "Oauth2"]
date = "2017-08-24T20:14:00-05:00"
description = "This time I will show you how to build a basic Spring Boot application with Google authentication by Oauth2."
+++

This time I will show you how to build a basic Spring Boot application with Google authentication using Oauth2.

**NOTE:** If you need to know what tools you need to have installed in yout computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot)

Then execute this command in your terminal:

```groovy
spring init --dependencies=web,security,thymeleaf --language=groovy --build=gradle spring-boot-oauth2
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

apply plugin: 'groovy'
apply plugin: 'org.springframework.boot'

version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
  mavenCentral()
}

dependencies {
  compile 'org.springframework.boot:spring-boot-starter-web'
  compile 'org.springframework.boot:spring-boot-starter-security'
  compile 'org.springframework.boot:spring-boot-starter-thymeleaf'
  compile 'org.codehaus.groovy:groovy'
  testCompile'org.springframework.boot:spring-boot-starter-test'
}
```

Next, we are going to add `security-oauth2` dependency:

```groovy
compile 'org.springframework.security.oauth:spring-security-oauth2'
```

Since we need to pass some options such as cliendId, clientSecret, etc. to the application the common way is to use `bootRun` task to specify them as system properties.

```
bootRun {
  systemProperties = System.properties
}
```

Not is time to create a configuration file, in this case we are going to use a yaml format. In your computer's home directory: ${home}, please create a directory called: `.oauth2` then inside create a file called `application-development.yml` with this content:

```yml
security:
  oauth2:
    client:
      clientId: clientId
      clientSecret: clientSecret
      accessTokenUri: https://www.googleapis.com/oauth2/v4/token
      userAuthorizationUri: https://accounts.google.com/o/oauth2/v2/auth
      clientAuthenticationScheme: form
      scope:
        - email
    resource:
      userInfoUri: https://www.googleapis.com/oauth2/v3/userinfo
```

In order to get an `clientId` and `clientSecret` you need to go to https://console.developers.google.com and login with your Google account, then in Credentials section create new Oauth client ID.

This is our `DemoApplication`

```groovy
package com.jos.dem.oauth2

import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.SpringBootApplication

@SpringBootApplication
class DemoApplication {

  static void main(String[] args) {
    SpringApplication.run DemoApplication, args
  }

}
```

And this is the `SecurityConfiguration`

```groovy
package com.jos.dem.oauth2.config

import org.springframework.context.annotation.Configuration
import org.springframework.security.config.annotation.web.builders.HttpSecurity
import org.springframework.boot.autoconfigure.security.oauth2.client.EnableOAuth2Sso
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter

@Configuration
@EnableOAuth2Sso
class SecurityConfig extends WebSecurityConfigurerAdapter {

  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http
    .antMatcher("/**")
    .authorizeRequests()
    .antMatchers("/")
    .permitAll()
    .anyRequest()
    .authenticated();
  }

}
```

`@EnableOAuth2Sso` Manages Oauth2 client and authentication. All we need to do to make our home page visible is to explicitly `authorizeRequests()`. All other requests (e.g. to the /user endpoint) require authentication.

This is our default controller

```groovy
package com.jos.dem.oauth2.controller

import org.springframework.stereotype.Controller
import org.springframework.web.bind.annotation.RequestMapping

@Controller
class DemoController {

  @RequestMapping('/')
  String index(){
    'index'
  }

}
```

And this our `index.html`

```html
<html>
  <body>
    <a th:href="@{/user/show}">Login using Google</a>
  </body>
</html>
```

All have access to this page, but as you can see we are redirecting to a secured web page. This is when the application asks to Google for authentication. Google reponse includes a `Principal` object with user's data information.

```groovy
package com.jos.dem.oauth2.controller

import java.security.Principal

import org.springframework.stereotype.Controller
import org.springframework.web.servlet.ModelAndView
import org.springframework.web.bind.annotation.RequestMapping

@Controller
@RequestMapping("/user")
class UserController {

  @RequestMapping('/show')
  ModelAndView show(Principal principal){
    Map details = [:]
    details.name = principal.name
    details.email =  principal.userAuthentication?.details?.email
    ModelAndView modelAndView = new ModelAndView('user/show')
    modelAndView.addObject('details', details)
    modelAndView
  }

}
```

<img src="/img/techtalks/spring/oauth2_google.png">

This is the user web page showing data retieved by Google:

```html
<html>
  <body>
    <h3 th:text="${details.name}" />
    <p th:text="${details.email}" />
  </body>
</html>
```

<img src="/img/techtalks/spring/oauth2_google_user.png">

To download the project:

```bash
git clone https://github.com/josdem/spring-boot-oauth2.git
```

To run the project:

```bash
gradle -Dspring.config.location=$HOME/.oauth2/application-development.yml bootRun
```

[Return to the main article](/techtalk/spring#Spring_Boot)
