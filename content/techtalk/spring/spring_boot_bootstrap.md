+++
title = "Spring Boot Bootstrap"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology","Spring Boot","Bootstrap"]
date = "2018-02-20T15:07:04-06:00"
description = "In this technical post I will show you how to add Bootstrap to your Spring Boot project."
+++

[Boostrap](https://getbootstrap.com/) was originally created by a designer and a developer at Twitter, Bootstrap has become one of the most popular front-end frameworks and open source projects in the world.

In this technical post I will show you how to add Bootstrap to your Spring Boot project. In order to get the setup for this example, please refer my previous post [Spring Boot JPA](/techtalk/spring/spring_boot_jpa)

First you need to add Bootstrap dependency to your project, the way you should do it is add it as a [Bower](https://bower.io/) plugin to your `build.gradle` file in this way, you delegate to Gradle the responsability to orchestrate Bootstrap dependency management.

```groovy
plugins {
  id 'com.craigburke.bower-installer' version '2.5.1'
}

bower {
  installBase = 'src/main/resources/static/assets/third-party'
  'bootstrap'('3.3.7'){
    source '**'
  }
}
```

Next, we are going to add header and footer to a simple html page using [navbar](https://v4-alpha.getbootstrap.com/components/navbar/) component.

**Header**

```html
<nav class="navbar navbar-inverse navbar-fixed-top">
  <font color="white">Spring Boot</font>
</nav>
```

**Footer**

```html
<footer>
  <nav class="navbar navbar-inverse navbar-fixed-bottom">
    <a class="navbar-brand" href="https://github.com/josdem/spring-boot-jpa">josdem 2018</a>
  </nav>
</footer>
```

Do not forget to add Bootstrap style sheet in your `head` section

```html
<head>
  <link rel="stylesheet" th:href="@{/assets/third-party/bootstrap/dist/css/bootstrap.min.css}" />
</head>
```

Another component we are going to use is [Jumbotron](https://v4-alpha.getbootstrap.com/components/jumbotron/) which is a kind of viewport to showcase messages on your site.

```html
<div class="jumbotron">
  <div class="container">
    <h1>Welcome!</h1>
    <p>Spring Boot Workshop</p>
    <a href="https://github.com/josdem/spring-boot-jpa" class="btn btn-primary btn-lg" role="button">Learn more</a>
  </div>
</div>
```

This is a complete html example with header, footer, form and jumbotron integrated.

```html
<html>
<head>
  <link rel="stylesheet" th:href="@{/assets/third-party/bootstrap/dist/css/bootstrap.min.css}" />
</head>
<body>
  <nav class="navbar navbar-inverse navbar-fixed-top">
    <font color="white">Spring Boot</font>
  </nav>
  <div class="jumbotron">
    <div class="container">
      <h1>Welcome!</h1>
      <p>Spring Boot Workshop</p>
      <a href="https://github.com/josdem/spring-boot-jpa" class="btn btn-primary btn-lg" role="button">Learn more</a>
    </div>
  </div>
  <div class="container">
  <form id="create" th:action="@{/persons}" th:object="${personCommand}" method="post">
    <div class="form-group">
  	  <label class="col-sm-1 col-form-label-lg" for="nickname">Nickname:</label>
  	  <input class="form-control form-control-lg" type="text" name="nickname" th:field="*{nickname}" placeholder="nickname" id="nickname"/>
  	  <label th:if="${#fields.hasErrors('nickname')}" th:errors="*{nickname}"></label>
    </div>
  	<br/>
    <div class="form-group">
  	  <label class="col-sm-1 col-form-label-lg" for="email">Email:</label>
  	  <input class="form-control form-control-lg" type="text" name="email" th:field="*{email}" placeholder="email" id="email"/>
  	  <label th:if="${#fields.hasErrors('email')}" th:errors="*{email}"></label>
    </div>
  	<br/><br/>
  	<button class="btn btn-success" id="btn-success" type="submit">Submit</button>
  </form>
  </div>
  <footer>
    <nav class="navbar navbar-inverse navbar-fixed-bottom">
      <a class="navbar-brand" href="https://github.com/josdem/spring-boot-jpa">josdem 2018</a>
    </nav>
  </footer>
</body>
</html>
```

Here is how it looks like:

<img src="/img/techtalks/spring/bootstrap.png"/>

For more information regarding to [Bootstrap components](https://getbootstrap.com/docs/3.3/components/) please visit this official documentation.


To Download the Project:

```bash
git clone https://github.com/josdem/spring-boot-jpa.git
git fetch
git checkout feature/bootstrap
```

To Run the project:

```bash
gradle bootRun
```

[Return to the main article](/techtalk/spring#Spring_Boot)
