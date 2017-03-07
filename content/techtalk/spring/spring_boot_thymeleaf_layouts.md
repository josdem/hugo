+++
tags = ["josdem","techtalks","programming","technology"]
date = "2017-03-07T13:10:45-06:00"
title = "Spring Boot Thymeleaf Layouts"
categories = ["techtalk","code"]

+++

This time I will show you how to share common page components like header, footer called layouts using Thymeleaf. Go [here](http://www.thymeleaf.org/doc/articles/layouts.html) for more information. Let's consider the following home page:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="stylesheet" href="/assets/third-party/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="/assets/third-party/font-awesome/font-awesome.less" />
  </head>
  <body>
    <nav class="navbar navbar-inverse navbar-fixed-top">
      <a class="navbar-brand" href="/">
        <img src="/assets/third-party/vetlog-theme/img/flex-admin-logo.png" th:src="@{/assets/third-party/vetlog-theme/img/flex-admin-logo.png}"/>
      </a>
    </nav>
    <nav class="navbar-top" role="navigation">
      <div class="navbar-brand">
      </div>
    </nav>
    <div class="jumbotron">
      <div class="container">
        <h1>Welcome!</h1>
        <p>Maintain your pet history organized</p>
        <p><a class="btn btn-primary btn-lg" href="login" role="button">Login Here</a></p>
      </div>
    </div>
    <div class="container">
      <div class="row">
        <div class="col-md-4">
          <h4>Register</h4>
          <p>Register all your pets, so you can check its healty care historial.</p>
          <a th:href="@{pet/create}" class="btn btn-success"><p th:text="#{button.action}"/></a>
        </div>
        <div class="col-md-4">
          <h4>Consult</h4>
          <p>Consult your pet historial, so you can keep up to date with its care.</p>
          <a class="btn btn-success" href="command-line" role="button">View details</a>
        </div>
        <div class="col-md-4">
          <h4>Adoption</h4>
          <p>Adopting a pet is a big step, find the right pet since it would be a member of your family.</p>
          <a class="btn btn-success" href="command-line" role="button">View details</a>
        </div>
      </div>
    </div>
    <footer>
      <nav class="navbar navbar-inverse navbar-fixed-bottom">
        <a class="navbar-brand" href="http://josdem.io">© josdem.io 2017</a>
      </nav>
    </footer>
  </body>
</html>
```

As you can see, in a single index page we are adding several elements such as links to the stylesheets, a header with navigation bar, content and footer. We can simplify this page adding layouts in separate files so we can reuse pieces of html.

${PROJECT_HOME}/resources/templates/fragments/include.html

```html
<head> 
  <link rel="stylesheet" href="/assets/third-party/bootstrap/dist/css/bootstrap.min.css" />
  <link rel="stylesheet" href="/assets/third-party/font-awesome/font-awesome.less" />
</head>
```

${PROJECT_HOME}/resources/templates/fragments/header.html

```html
<body>
  <nav class="navbar navbar-inverse navbar-fixed-top">
    <a class="navbar-brand" href="/">
      <img src="/assets/third-party/vetlog-theme/img/flex-admin-logo.png" th:src="@{/assets/third-party/vetlog-theme/img/flex-admin-logo.png}"/>
    </a>
  </nav>
  <nav class="navbar-top" role="navigation">
    <div class="navbar-brand">
    </div>
  </nav>
  <div class="jumbotron">
    <div class="container">
      <h1>Welcome!</h1>
      <p>Maintain your pet history organized</p>
      <p><a class="btn btn-primary btn-lg" href="login" role="button">Login Here</a></p>
    </div>
  </div>
</body>
```

${PROJECT_HOME}/resources/templates/fragments/footer.html

```html
<footer>   
	<nav class="navbar navbar-inverse navbar-fixed-bottom">
		<a class="navbar-brand" href="http://josdem.io">© josdem.io 2017</a>
	</nav>
</footer>
```

So we can reuse html code from our layout files using `<th:block layout:include="fragments/include"/>` In this case, the entire fragments/include.html template will be replaced by th block.

${PROJECT_HOME}/resources/templates/home/home.html

```html
<!DOCTYPE html>
<html>
<head>
<th:block layout:include="fragments/include"/>
</head>
<body>
  <th:block layout:include="fragments/header"/>
  <div class="container">
    <div class="row">
      <div class="col-md-4">
        <h4>Register</h4>
        <p>Register all your pets, so you can check its healty care historial.</p>
        <a th:href="@{pet/create}" class="btn btn-success"><p th:text="#{button.action}"/></a>
      </div>
      <div class="col-md-4">
        <h4>Consult</h4>
        <p>Consult your pet historial, so you can keep up to date with its care.</p>
        <a class="btn btn-success" href="command-line" role="button">View details</a>
      </div>
      <div class="col-md-4">
        <h4>Adoption</h4>
        <p>Adopting a pet is a big step, find the right pet since it would be a member of your family.</p>
        <a class="btn btn-success" href="command-line" role="button">View details</a>
      </div>
    </div>
  </div>
  <th:block layout:include="fragments/footer"/>
</body>
</html>
```

To download the project:

```bash
git clone https://github.com/josdem/vetlog-spring-boot.git
git fetch
git checkout feature/34
```

To run the project, please read the [wiki](https://github.com/josdem/vetlog-spring-boot/wiki/YAML%20File) and execute this command:

```bash
gradle bootRun -Dspring.config.location=$HOME/.vetlog-spring-boot/application-development.yml
```

To run only this test:

```bash
gradle -Dtest.single=UserValidatorSpec test -Dspring.config.location=$HOME/.vetlog-spring-boot/application-development.yml
```

[Return to the main article](/techtalk/spring)

