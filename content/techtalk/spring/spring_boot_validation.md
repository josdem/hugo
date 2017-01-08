+++
categories = ["techtalk","code"]
tags = ["josdem","techtalks","programming","technology","javax validation"]
date = "2017-01-08T09:54:20-06:00"
title = "Spring Boot Validation"

+++

Spring has a `Validation` interface that we can use to validate objects, if some errors occur in the validation it store them in a `BindingResult` object so you can test for and retrieve validation errors. Let's consider the following validator:

```groovy
package com.jos.dem.vetlog.validator

import org.springframework.validation.Validator
import org.springframework.validation.Errors
import org.springframework.stereotype.Component

import com.jos.dem.vetlog.command.UserCommand

@Component
class UserValidator implements Validator {

  @Override
  boolean supports(Class<?> clazz) {
    UserCommand.class.equals(clazz)
  }

  @Override
  void validate(Object target, Errors errors) {
    UserCommand UserCommand = (UserCommand) target
  }

}
```

This class provides validation behaviour implementing `org.springframework.validation.Validator`

* `supports(Class)` Define which class can be validated
* `validate(Object, org.springframework.validation.Errors)` Object validation, if some errors occurs store them in `Errors`

```groovy
package com.jos.dem.vetlog.controller

import static org.springframework.web.bind.annotation.RequestMethod.GET
import static org.springframework.web.bind.annotation.RequestMethod.POST

import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.InitBinder
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.web.bind.WebDataBinder
import org.springframework.web.servlet.ModelAndView
import org.springframework.validation.BindingResult
import org.springframework.stereotype.Controller
import javax.validation.Valid

import com.jos.dem.vetlog.command.UserCommand
import com.jos.dem.vetlog.validator.UserValidator

@Controller
@RequestMapping("/users")
class UserController {

  @Autowired
  UserValidator userValidator

  @InitBinder
  private void initBinder(WebDataBinder binder) {
    binder.addValidators(userValidator)
  }

  @RequestMapping(method = GET, value = "/create")
  ModelAndView create(){
    def modelAndView = new ModelAndView('user/create')
    def userCommand = new UserCommand()
    modelAndView.addObject('userCommand', userCommand)
    modelAndView
  }

  @RequestMapping(method = POST, value = "/save")
  String save(@Valid UserCommand command, BindingResult bindingResult) {
    if (bindingResult.hasErrors()) {
      return 'user/create'
    }
    'redirect:/'
  }

}
```

You can retrieve all the attributes from the form bound to the `UserCommand` object. In the code, you test for errors, and if so, send the user back to the original form template. In that situation, all the error attributes are displayed.

If all of the user’s attribute are valid, then it redirects the browser to the final results template.

```html
<!DOCTYPE html>
<html>
<head>
  <link rel="stylesheet" href="/assets/third-party/bootstrap/dist/css/bootstrap.min.css" />
  <link rel="stylesheet" href="/assets/third-party/font-awesome/css/font-awesome.css" />
  <link rel="stylesheet" href="/assets/third-party/vetlog-theme/css/style.css" />
  <link rel="stylesheet" href="/assets/third-party/vetlog-theme/css/plugins.css" />
  <link rel="stylesheet" href="/assets/third-party/vetlog-theme/css/demo.css" />
  <title th:text="#{user.view.create.title}"></title>
</head>
<body class="login">
  <div class="container">
    <div class="row">
      <div class="col-md-6 col-md-offset-3">
        <div class="login-banner text-center">
          <img src="/assets/third-party/vetlog-theme/img/flex-admin-logo.png" th:src="@{/assets/third-party/vetlog-theme/img/flex-admin-logo.png}"/>
        </div>
        <div class="portlet portlet-green">
          <div class="portlet-heading login-heading">
            <div class="portlet-title">
              <h4 th:text="#{user.view.create.label}"/>
            </div>
            <div class="clearfix"></div>
          </div>
          <div class="portlet-body">
            <form id="register" action="#" th:action="@{/users/save}" th:object="${userCommand}" method="post"  class="form-horizontal">
              <fieldset>
                <label for="username"><h4 th:text="#{login.username}"/></label>
                <input type="text" name='username' class="form-control" placeholder="username" id='username'/>
                <label th:if="${#fields.hasErrors('username')}" th:errors="*{username}"></label>
                <br/>
                <label for="password"><h4 th:text="#{login.password}"/></label>
                <input type="password" name='password' class="form-control" placeholder="password" id='password'/>
                <label th:if="${#fields.hasErrors('password')}" th:errors="*{password}"></label>
                <br/>
                <label for="passwordConfirmation"><h4 th:text="#{login.passwordConfirmation}"/></label>
                <input type="password" name='passwordConfirmation' class="form-control" placeholder="password confirmation" id='passwordConfirmation'/>
                <label th:if="${#fields.hasErrors('passwordConfirmation')}" th:errors="*{passwordConfirmation}"></label>
                <br/>
                <label for="name"><h4 th:text="#{user.name}"/></label>
                <input type="text" name='name' class="form-control" placeholder="name" id='name'/>
                <label th:if="${#fields.hasErrors('name')}" th:errors="*{name}"></label>
                <br/>
                <label for="lastname"><h4 th:text="#{user.lastname}"/></label>
                <input type="text" name='lastname' class="form-control" placeholder="lastname" id='lastname'/>
                <label th:if="${#fields.hasErrors('lastname')}" th:errors="*{lastname}"></label>
                <br/>
                <label for="email"><h4 th:text="#{user.email}"/></label>
                <input type="text" name='email' class="form-control" placeholder="email" id='email'/>
                <label th:if="${#fields.hasErrors('email')}" th:errors="*{email}"></label>
                <br/>
                <input type="submit" class="btn btn-primary btn-block" />
              </fieldset>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</body>
</html>
```

The page contains a simple form. It is marked as being backed up by the user object that you saw in the GET method in the web controller. This is known as a bean-backed form. You can see them tagged `th:field="{username}"`, etc. Next to each field is a secondary element used to show any validation errors.

```groovy
package com.jos.dem.vetlog.command

import javax.validation.constraints.NotNull
import javax.validation.constraints.Size
import org.hibernate.validator.constraints.Email

class UserCommand implements Command {

  @NotNull
  @Size(min=6, max=50)
  String username

  @NotNull
  @Size(min=8, max=50)
  String password

  @NotNull
  @Size(min=8, max=50)
  String passwordConfirmation

  @NotNull
  @Size(min=1, max=50)
  String name

  @NotNull
  @Size(min=1, max=100)
  String lastname

  @Email
  String email
}
```

In the model `UserCommand` we defined all fields as not empty, size min and max and email validation format using hibernate validator.

To download the project

```bash
git clone https://github.com/josdem/vetlog-spring-boot.git
git fetch
git checkout feature/3
```

To run the project:

```bash
gradle bootRun
```

[Return to the main article](/techtalk/spring)
