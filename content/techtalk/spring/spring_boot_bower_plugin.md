+++
title = "Spring Boot Bower Plugin"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology","bower","gradle","spring boot"]
date = "2017-01-01T15:00:01-06:00"
description = "Applying a plugin to a project allows the plugin to extend the project's capabilities. In this example I will apply Bower plugin to a Spring Boot project."
+++

Applying a plugin to a project allows the plugin to extend the project's capabilities. In this example I will apply Bower plugin to a Spring Boot project. Bower is a package manager for the web specifically HTML, Javascript libraries, CSS fonts even image files. Look here for more information: [Bower](https://bower.io/)

In order to add bower plugin, we need to modify the `build.gradle` file adding this lines:

```groovy
plugins {
  id 'com.craigburke.bower-installer' version '2.5.1'
}
```

You can find more information about bower plugin [here](https://github.com/craigburke/bower-installer-gradle)

The plugin adds the `bowerInstall` task to your build, so this task install all bower dependencies that you have set in your `build.gradle`. In this example I will add the Bootstrap dependency. Bootstrap is the most popular HTML, CSS, and JS framework for developing responsive, mobile first projects on the web. Look here for more information: [Boostrap](http://getbootstrap.com/)

```groovy
bower {
  installBase = 'src/main/resources/static/assets/third-party'
  'bootstrap'('3.3.7'){
    source '**'
  }
}
```

Finally we are going to specify `bootRun` task that depends on `bowerInstall` task. That's it, before gradle executes our project will download all bower dependencies to our project in the `src/main/resources/static/assets/third-party` path


```groovy
bootRun.dependsOn rootProject.tasks['bowerInstall']
```

To download the project

```bash
git clone https://github.com/josdem/vetlog-spring-boot.git
git fetch
git checkout feature/bower-plugin
```

To run the project:

```bash
gradle bootRun
```

[Return to the main article](/techtalk/spring#Spring_Boot)
