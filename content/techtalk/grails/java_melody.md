+++
title =  "Java Melody"
date = "2018-05-13T13:46:04-05:00"
tags = ["josdem", "techtalks","programming","technology", "java melody", "java melody grails configuration"]
categories = ["techtalk", "code", "grails"]
description = "The goal of JavaMelody is to monitor Java applications in production environments. It is a tool to measure and calculate statistics on real operation of an application depending on the usage of the application by users."
+++

The goal of [JavaMelody](https://github.com/javamelody/javamelody) is to monitor Java applications in production environments. It is a tool to measure and calculate statistics on real operation of an application depending on the usage of the application by users.

This post go through the configuration needed in a Grails application, in this post we are using Grails 3.3.5 version. First let's create a new fresh Grails project:

```bash
grails create-app grails-java-melody
```

**NOTE:** If you need to know what tools you need to have installed in yout computer in order to create a Grails basic project, please refer my previous post: [Grails Hello World](/techtalk/grails/hello_world)

**Installation**

To install the plugin, just add Grails dependency in your `build.gradle` file:

```groovy
dependencies {
  compile 'org.grails.plugins:grails-melody-plugin:1.72.0'
}
```

Then you will be able to monitor the application at [http://localhost:8080/monitoring](http://localhost:8080/monitoring).

**Result**

<img src="/img/techtalks/grails/java_melody.png">

**More configuration**

If you need to disable JavaMelody monitoring add this lines to your `grails-app/conf/application.yml`:

```yaml
javamelody:
  disabled: true
```

Also all parameters described in the [JavaMelody User's guide](https://github.com/javamelody/javamelody/wiki/UserGuide#6-optional-parameters) can be configured in your `grails-app/conf/application.yml` file. You can also add rules in your `grails-app/conf/application.groovy` for the [spring security plugin](https://grails-plugins.github.io/grails-spring-security-core/latest/), if installed:

```groovy
grails.plugin.springsecurity.controllerAnnotations.staticRules = [ [pattern: '/monitoring', access: ['ROLE_ADMIN']] ]
```

To download the project:

```bash
git clone https://github.com/josdem/grails-java-melody.git
```

To run the project:

```bash
grails run-app
```

[Return to the main article](/techtalk/grails)