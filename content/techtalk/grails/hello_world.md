+++
title =  "Grails Hello World"
description = "Grails Framework is an open source web application framework that uses the Apache Groovy programming language. It is intended to be a high-productivity framework by following the coding by convention paradigm, providing a stand-alone development environment and hiding much of the configuration detail from the developer"
date = "2018-05-11T10:51:53-05:00"
tags = ["josdem", "techtalks","programming","technology", "grails"]
categories = ["techtalk", "code", "grails"]
+++


[Grails Framework](https://grails.org/) is an open source web application framework that uses the [Apache Groovy](http://groovy-lang.org/) programming language. It is intended to be a high-productivity framework by following the coding by convention paradigm, providing a stand-alone development environment and hiding much of the configuration detail from the developer.

**Setup**

This time I will show you how to create a basic project in Grails, in order to do that you should install [SDKMAN](http://sdkman.io/) if you are using Linux or Mac, or [posh-gvm](https://github.com/flofreud/posh-gvm) if you are using Windows. After that, you can easily install:

* Groovy
* Gradle
* Grails

**Manual Setup**

You can also download binary distributions (Groovy, Gradle and Grails) and setting `GROOVY_HOME`, `GRADLE_HOME` and `GRAILS_HOME` in your local computer environment variables.


Then execute this command in your terminal:

```bash
grails create-app hello-world
```

This will create a new directory inside the current one that contains the project. This is an example about this project directory structure:

```
grails-hello-world
├── build.gradle
├── gradle
│   └── wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradle.properties
├── gradlew
├── gradlew.bat
├── grails-app
│   ├── assets
│   ├── conf
│   │   ├── application.yml
│   │   ├── logback.groovy
│   │   └── spring
│   │       └── resources.groovy
│   ├── controllers
│   │   └── hello
│   │       └── world
│   │           └── UrlMappings.groovy
│   ├── i18n
│   ├── init
│   │   └── hello
│   │       └── world
│   │           ├── Application.groovy
│   │           └── BootStrap.groovy
│   └── views
│       ├── error.gsp
│       ├── index.gsp
│       ├── layouts
│       │   └── main.gsp
│       └── notFound.gsp
├── grailsw
├── grailsw.bat
├── grails-wrapper.jar
└── src
    └── integration-test
        └── resources
            └── GebConfig.groovy

26 directories, 65 files
```

**Convention over Configuration**

Instead of configuration Grails use specific names and locations to store files in a structure following a convention:

* `build.gradle` Build, plugins and dependencies specification.
* `grails-app` Source code structure
	*  `assets` Images, JavaScript and Stylesheets
	* `conf` Application configuration 
	* `controllers` Web MVC controllers
	* `services` Business layer services
	* `domain` Application model 
	* `view` Groovy Server Pages
	* `i18n` Internationalization files
* `src` Groovy and Java files including test	

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

**Creating Controller**
 
Let’s now create Person controller with the `create-controller` command:

```bash
grails> generate-controller com.jos.dem.Person
```
output:

```bash
| Rendered template Controller.groovy to destination grails-app\controllers\com\jos\dem\PersonController.groovy
| Rendered template Service.groovy to destination grails-app\services\com\jos\dem\PersonService.groovy
| Rendered template Spec.groovy to destination src\test\groovy\com\jos\dem\PersonControllerSpec.groovy
| Rendered template ServiceSpec.groovy to destination src\integration-test\groovy\com\jos\dem\PersonServiceSpec.groovy
| Scaffolding completed for grails-app\domain\com\jos\dem\Person.groovy
```

Previous command is using static scaffolding, it lets you generate a controller based in templates and it provides you necessary code to create CRUD (Create/Read/Update/Delete) operations over Person model. 

```groovy
package com.jos.dem

import grails.validation.ValidationException
import static org.springframework.http.HttpStatus.*

class PersonController {

    PersonService personService

    static allowedMethods = [save: "POST", update: "PUT", delete: "DELETE"]

    def index(Integer max) {
        params.max = Math.min(max ?: 10, 100)
        respond personService.list(params), model:[personCount: personService.count()]
    }

    def show(Long id) {
        respond personService.get(id)
    }

    def create() {
        respond new Person(params)
    }

    def save(Person person) {
        if (person == null) {
            notFound()
            return
        }

        try {
            personService.save(person)
        } catch (ValidationException e) {
            respond person.errors, view:'create'
            return
        }

        request.withFormat {
            form multipartForm {
                flash.message = message(code: 'default.created.message', args: [message(code: 'person.label', default: 'Person'), person.id])
                redirect person
            }
            '*' { respond person, [status: CREATED] }
        }
    }

    def edit(Long id) {
        respond personService.get(id)
    }

    def update(Person person) {
        if (person == null) {
            notFound()
            return
        }

        try {
            personService.save(person)
        } catch (ValidationException e) {
            respond person.errors, view:'edit'
            return
        }

        request.withFormat {
            form multipartForm {
                flash.message = message(code: 'default.updated.message', args: [message(code: 'person.label', default: 'Person'), person.id])
                redirect person
            }
            '*'{ respond person, [status: OK] }
        }
    }

    def delete(Long id) {
        if (id == null) {
            notFound()
            return
        }

        personService.delete(id)

        request.withFormat {
            form multipartForm {
                flash.message = message(code: 'default.deleted.message', args: [message(code: 'person.label', default: 'Person'), id])
                redirect action:"index", method:"GET"
            }
            '*'{ render status: NO_CONTENT }
        }
    }

    protected void notFound() {
        request.withFormat {
            form multipartForm {
                flash.message = message(code: 'default.not.found.message', args: [message(code: 'person.label', default: 'Person'), params.id])
                redirect action: "index", method: "GET"
            }
            '*'{ render status: NOT_FOUND }
        }
    }
}
```

**Creating Views**

Now is time to generate GSP (Groovy Server Pages) views for the given Person domain class. Using also static scaffolding the command will generate the appropriate 'list', 'show', 'create' and 'edit' views in `grails-app/views/person`.

```bash
grails> generate-views com.jos.dem.Person
```

output:

```bash
| Rendered template create.gsp to destination grails-app\views\person\create.gsp
| Rendered template edit.gsp to destination grails-app\views\person\edit.gsp
| Rendered template index.gsp to destination grails-app\views\person\index.gsp
| Rendered template show.gsp to destination grails-app\views\person\show.gsp
| Views generated for grails-app\domain\com\jos\dem\Person.groovy
```

Finally let's execute our application using this command:

```bash
grails> run-app
```

Going to `http://localhost:8080/` you can see a link to Person controller where you can do CRUD operations over Person.

<img src="/img/techtalks/grails/hello_world.png">


To download the project:

```bash
git clone https://github.com/josdem/grails-hello-world.git
```

To run the project:

```bash
grails run-app
```

[Return to the main article](/techtalk/grails)