+++
title =  "Grails and Bower Integration"
description = "Grails 2.5.6 and Bower integration using Asset pipeline plugin to manage asset dependencies"
date = "2018-05-29T11:18:31-05:00"
tags = ["josdem", "techtalks","programming","technology","grails 2.5.6", "grails bower integration"]
categories = ["techtalk", "code", "grails"]
+++

[Asset Pipeline Plugin](http://plugins.grails.org/plugin/grails/asset-pipeline) allows you to do assets management in a Grails application, it is highly extensible and provides processing of dynamic languages like LESS, SASS, Coffee, Typescript, and more. In another hand [Bower](https://bower.io/) allows you to download assets, libraries, javascripts frameworks and utilites. This time I will show you how to manage assets dependencies using Asset pipeline plugin with Bower in a Grails 2.5.6 project.

**NOTE:** If you want to know what tools you need to have installed in yout computer in order to create a Grails basic project, please refer my previous post: [Grails Hello World](/techtalk/grails/hello_world)

**Creating Model**

Model is cornerstone in our application it defines business model representation and usually represent persistent data base entities using underhood Hibernate and ORM (Object-Relational mapping).

Let's create model in our project, so within project directory you just created, start Grails interactive console:

```bash
grails
```
Then in order to create a domain named Person type this command in your command line:

```bash
grails> create-domain-class com.jos.dem.Person
```

Output:

```bash
| Created grails-app/domain/com/jos/dem/Person.groovy
| Created src/test/groovy/com/jos/dem/PersonSpec.groovy
```

Grails uses static constraints in order to create model validations, this is build on Spring’s validator API. This is our `Person.groovy` model created and with some basic constraints validations:

```groovy
package com.jos.dem

class Person {
  String nickname
  String email

  static constraints = {
    nickname blank:false,size:3..50
    email blank:false,size:8..250
  }
}
```

Now Grails needs to download some plugins and dependencies, let's start our Grails application before continue with next steps.

```bash
grails> run-app
```

Once your Grails proyect is up an running you will be able to create a Person's controller and Person's views, please refer my previous post: [Grails Hello World](/techtalk/grails/hello_world) for more information. At this point you can do CRUD operations over Person model. So let's continue installing Bower, as requirement you also need install [nodejs](https://nodejs.org/en/) so follow installation instructions and proceed forming `bower.json` file structure as follow:

```bash
bower init
```

Provide some basic required information about your project and at the end of the process you will get a similar file like this one:

```json
{
  "name": "grails2-bower-workshop",
  "description": "Grails2 application and bower integrarion demo",
  "main": "",
  "authors": [
    "joseluis.delacruz@gmail.com"
  ],
  "license": "MIT",
  "homepage": "https://github.com/josdem/grails2-bower-workshop",
  "ignore": [
    "**/.*",
    "node_modules",
    "bower_components",
    "test",
    "tests"
  ]
}
```

Next step is provide to Bower dependencies we need, so add `dependencies` section to your Bower file, in this case we need bootstrap and datepicker:

```json
{
  "name": "grails2-bower-workshop",
  "description": "Grails2 application and bower integrarion demo",
  "main": "",
  "authors": [
    "joseluis.delacruz@gmail.com"
  ],
  "license": "MIT",
  "homepage": "https://github.com/josdem/grails2-bower-workshop",
  "ignore": [
    "**/.*",
    "node_modules",
    "bower_components",
    "test",
    "tests"
  ],
  "dependencies": {
    "bootstrap": "3.3.7",
    "bootstrap-datepicker": "1.8.0"
  }
}
```

After this, if you execute in your command line: `bower install` a directory called `bower_components` will be created with bootstrap 3.3.7 and bootstrap-picker 1.8.0, but Asset pipeline will look for files in `grails-app/assets`. In order to change Bower target path we need to create a new file in our project root named: `.bowerrc` with this content:

```json
{
  "directory": "grails-app/assets/javascripts/third-party"
}
```

That's it, now Bower will not create a `bower_component` directory but a `grails-app/assets/javascripts/third-party` target directory instead. Now it is time to avoid Grails load all javascript files and only load what we need, so open `grails-app/assets/javascripts/application.js` and delete last lines: //= require jquery, //= require_tree . and //= require_self.


```javascript
// This is a manifest file that'll be compiled into application.js.
//
// Any JavaScript file within this directory can be referenced here using a relative path.
//
// You're free to add application-wide JavaScript to this file, but it's generally better 
// to create separate JavaScript files as needed.
//
//= require jquery
//= require_tree .
//= require_self
```

Let's open `grails-app/views/person/create.gsp` and add css and javascript link references to the downloaded files by Bower:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta name="layout" content="main">
    <g:set var="entityName" value="${message(code: 'person.label', default: 'Person')}" />
    <title><g:message code="default.create.label" args="[entityName]" /></title>
    <asset:stylesheet src="third-party/bootstrap-datepicker/dist/css/bootstrap-datepicker.css"/>    
    <asset:javascript src="third-party/jquery/dist/jquery.js"/>
    <asset:javascript src="third-party/bootstrap-datepicker/dist/js/bootstrap-datepicker.js"/>    
  </head>
  <body>
    <a href="#create-person" class="skip" tabindex="-1"><g:message code="default.link.skip.label" default="Skip to content&hellip;"/></a>
    <div class="nav" role="navigation">
      <ul>
        <li><a class="home" href="${createLink(uri: '/')}"><g:message code="default.home.label"/></a></li>
        <li><g:link class="list" action="index"><g:message code="default.list.label" args="[entityName]" /></g:link></li>
      </ul>
    </div>
    <div id="create-person" class="content scaffold-create" role="main">
      <h1><g:message code="default.create.label" args="[entityName]" /></h1>
      <g:if test="${flash.message}">
      <div class="message" role="status">${flash.message}</div>
      </g:if>
      <g:hasErrors bean="${personInstance}">
      <ul class="errors" role="alert">
        <g:eachError bean="${personInstance}" var="error">
        <li <g:if test="${error in org.springframework.validation.FieldError}">data-field-id="${error.field}"</g:if>><g:message error="${error}"/></li>
        </g:eachError>
      </ul>
      </g:hasErrors>
      <g:form url="[resource:personInstance, action:'save']" >
        <fieldset class="form">
          <g:render template="form"/>
        </fieldset>
        <fieldset class="buttons">
          <g:submitButton name="create" class="save" value="${message(code: 'default.button.create.label', default: 'Create')}" />
        </fieldset>
      </g:form>
    </div>    
    <asset:javascript src="datepicker.js"/>   
  </body>
</html>
```

As a final step, we need to create a `g:textField` html component with `datepicker` id so we can use bootstrap-datepicker:

```html
<%@ page import="com.jos.dem.Person" %>

<div class="fieldcontain ${hasErrors(bean: personInstance, field: 'nickname', 'error')} required">
  <label for="nickname">
    <g:message code="person.nickname.label" default="Nickname" />
    <span class="required-indicator">*</span>
  </label>
  <g:textField name="nickname" maxlength="50" required="" value="${personInstance?.nickname}"/>

</div>

<div class="fieldcontain ${hasErrors(bean: personInstance, field: 'email', 'error')} required">
  <label for="email">
    <g:message code="person.email.label" default="Email" />
    <span class="required-indicator">*</span>
  </label>
  <g:textField name="email" maxlength="250" required="" value="${personInstance?.email}"/>

</div>

<div class="fieldcontain">
  <label for="birthdate">
    <g:message code="person.birthdate.label" default="Birthdate" />   
  </label>
  <g:textField id='datepicker' name="birthDate" maxlength="250"/>
</div>
```

This is our `grails-app/assets/javascripts/datepicker.js` file:

```javascript
$('#datepicker').datepicker()
```

Now when you run your Grails project and create a new Person you will be able to use datepicker calendar component, go [here](https://bootstrap-datepicker.readthedocs.io/en/latest/) for more information how to use it.

To download the project:

```bash
git clone https://github.com/josdem/grails2-bower-workshop.git
```

To run the project:

```bash
grails run-app
```

[Return to the main article](/techtalk/grails)


