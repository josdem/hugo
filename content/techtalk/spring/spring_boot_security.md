+++
date = "2017-01-10T19:25:41-06:00"
title = "Spring Boot Security"
tags = ["josdem","techtalks","programming","technology","spring boot"]
categories = ["techtalk","code"]

+++

Spring Security is a powerful and highly customizable authentication and access-control framework. In this example I will show you how to integrate it to your Spring Boot project. First we need to add dependency to the `build.gradle`

```groovy
compile 'org.springframework.boot:spring-boot-starter-security'
```

Then we need to configure Spring secutiry using Java config class:

```groovy
package com.jos.dem.vetlog.config

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.context.annotation.Configuration
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder
import org.springframework.security.config.annotation.web.builders.HttpSecurity
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity

@Configuration
@EnableWebSecurity
class SecurityConfig extends WebSecurityConfigurerAdapter {
  @Override
  protected void configure(HttpSecurity http) throws Exception {
    http
    .authorizeRequests()
    .antMatchers("/", "/assets/**","/home/**").permitAll()
    .anyRequest().authenticated()
    .and()
    .formLogin()
    .loginPage("/login")
    .permitAll()
    .and()
    .logout()
    .permitAll()
  }

  @Autowired
  void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
    auth
    .inMemoryAuthentication()
    .withUser("user").password("password").roles("USER")
  }
}
```

The `SecurityConfig` class is annotated with `@EnableWebSecurity` to enable Spring Security’s web security support and provide the Spring MVC integration.

The `configure(HttpSecurity)` method defines which URL paths should be secured and which should not. Specifically, the `"/"`, `"/home"` and `"/assets"` paths are configured to not require any authentication. All other paths must be authenticated.

When a user successfully logs in, they will be redirected to the previously requested page that required authentication. There is a custom `"/login"` page specified by `loginPage()`, and everyone is allowed to view it.

As for the `configureGlobal(AuthenticationManagerBuilder)` method, it sets up an in-memory user store with a single user. That user is given a username of `"user"`, a password of `"password"`, and a role of `"USER"`.

Now we need to create the login page. There’s already a view controller for the `"login"` view, so you only need to create the login view itself:

```html
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="/assets/third-party/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="/assets/third-party/font-awesome/css/font-awesome.css" />
    <link rel="stylesheet" href="/assets/third-party/vetlog-theme/css/style.css" />
    <link rel="stylesheet" href="/assets/third-party/vetlog-theme/css/plugins.css" />
    <link rel="stylesheet" href="/assets/third-party/vetlog-theme/css/demo.css" />
  </head>
  <body class="login">

    <div class="container">
      <div class="row">
        <div class="col-md-4 col-md-offset-4">
          <div class="login-banner text-center">
            <h1>
              <img src="/assets/third-party/vetlog-theme/img/flex-admin-logo.png" th:src="@{/assets/third-party/vetlog-theme/img/flex-admin-logo.png}"/>
            </h1>
          </div>
          <div class="portlet portlet-green">
            <div class="portlet-heading login-heading">
              <div class="portlet-title">
                <h4 th:text="#{login.welcome}"/>
              </div>
              <div class="clearfix"></div>
            </div>
            <div class="portlet-body">
              <div th:if="${error}">
                <div align="center">
                  <p th:text="${error}"/>
                </div>
              </div>
              <form th:action="@{/login}" method='POST' id='loginForm' class='cssform' autocomplete='off'>
                <fieldset>
                  <label for="username"><h4 th:text="#{login.username}"/></label>
                  <input type="text" name='username' class="form-control" placeholder="username" id='username'/>
                  <label for="password"><h4 th:text="#{login.password}"/></label>
                  <input type="password" name='password' class="form-control" placeholder="password" id='password'/>
                  <br/>
                  <button id="btn-success" type="submit" class="btn btn-lg btn-primary btn-block"><h5 th:text="#{login.action}"/></button>
                  <hr/>
                  <a th:href="@{'/users/create'}" class="btn btn-primary btn-block"><h5 th:text="#{login.register}"/></a>
                </fieldset>
                <br/>
                <p class="small">
                <g:link controller="recovery" action="forgotPassword"><h5 th:text="#{login.forgot}"/></g:link>
                </p>
              </form>
            </div>
          </div>
        </div>
      </div>
    </div>
  </body>
</html>
```

Finally in the `LoginController` we render error object message in case that the attempt to login failed and the view is specified.


```groovy
package com.jos.dem.vetlog.controller

import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RequestParam
import org.springframework.web.servlet.ModelAndView
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.beans.factory.annotation.Value
import org.springframework.stereotype.Controller

import com.jos.dem.vetlog.service.LocaleService
import com.jos.dem.vetlog.service.VetlogService

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@Controller
class LoginController {

  @Autowired
  LocaleService localeService

  Logger log = LoggerFactory.getLogger(this.class)

  @RequestMapping("/login")
  ModelAndView login(@RequestParam Optional<String> error){
    log.info "Calling login"
    ModelAndView modelAndView = new ModelAndView('login/login')
    if(error.isPresent()){
      modelAndView.addObject('error', localeService.getMessage('login.error'))
    }
    modelAndView
  }

}
```

To download the project

```bash
git clone https://github.com/josdem/vetlog-spring-boot.git
git fetch
git checkout feature/6
```

To run the project:

```bash
gradle bootRun
```

[Return to the main article](/techtalk/spring)

