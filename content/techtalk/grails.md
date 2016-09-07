+++
date = "2015-08-29T18:06:10-05:00"
draft = true
title = "Grails"

+++

Grails is an open source web application framework it is intended to be a high-productivity framework by following the "coding by convention" paradigm and ready-to-use development environment

## Characteristics

* Web Framework
* Open Source
* Cross-platform
* MVC paradigm

## Features

* Convention-over-Configuration
* Ready-to-use development environment
* Functionality available through mixins
* GSP technology
* Integrated ORM
* DRY (Don’t repeat yourself)
* Fullstack framework
* Scripting with Gant
* Tomcat application server embedded
* Transactional layout
* Plugin oriented
* Wide IDE Support
* Expressive Domain Specific Language
* Spring support
* Highly productive

## Creating a simple structure in Grails
Creating an application
In order to create an application type this command in your command line:

```bash
grails create-app project-creator
```

Output:

```bash
Created Grails Application at /home/josdem/Temporal/Grails/project-creator
```

Grails will create an directory named as the application name, go to that directory and continue creating a domain

## Creating a domain
Grails create a relation between domain classes and a table in a database, this is known as ORM (Object-relational Mapping) and Grails uses Hibernate for this.

In order to create a domain named Project type this command in your command line:

```bash
grails create-domain-class com.josdem.Project
```

Output:

```bash
Created file grails-app/domain/com/josdem/Project.groovy
Created file test/unit/com/josdem/ProjectSpec.groovy
```

Grails validation capability is build on Spring’s validator API. However Grails takes this further and provides a unified way to define validation "constraints" with its constraints mechanism.

```groovy
package com.josdem

class Project {
  String name

  static constraints = {
    name blank:false,size:1..15
  }
}
```

In this case we are validating in Project that name can not be blank and its size is between 1 and 15 characters

## Scaffolding
Scaffolding lets you generate some basic CRUD interfaces for a domain class, including:

* The necessary views
* Controller actions for create/read/update/delete (CRUD) operations

## Creating a controller
In order to create a controller we need to start grails application, type this:

```bash
grails
```

And then:

```bash
run-app
```

Output:

```bash
Server running. Browse to http://localhost:8080/project-creator
Application loaded in interactive mode. Type 'stop-app' to shutdown.
Enter a script name to run. Use TAB for completion:
```

Now we can create a controller named ProjectController type this command in your command line:

```bash
create-scaffold-controller com.josdem.Project
```

Output:

```bash
Created file grails-app/controllers/com/josdem/ProjectController.groovy
Created file grails-app/views/project
Created file test/unit/com/josdem/ProjectControllerSpec.groovy
```

Now you can explore your the application and the scaffolding

* [Download counter](/techtalk/grails/operating_system_downloader_counter)
* [Externalized configuration](/techtalk/grails/grails_externalized_conf)
* [Login functionality with Spring Security](/techtalk/grails/spring_security_login)

[Return to the main article](/techtalk/techtalks)
