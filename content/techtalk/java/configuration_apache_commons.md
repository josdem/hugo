+++
date = "2017-01-17T18:55:39-06:00"
title = "Configuration with Apache Commons"
categories = ["techtalk","code","java"]
tags = ["josdem","techtalks","programming","technology","java"]
description = "The Commons Configuration software library provides a generic configuration interface which enables a Java application to read configuration data from a variety of sources."
+++

The Commons Configuration software library provides a generic configuration interface which enables a Java application to read configuration data from a variety of sources. Go here for more information about [Apache Commons](http://commons.apache.org/)

In this post I will show you how to read a properties file using Apache commons. First we need to create a Java basic project this time using [lazybones](https://github.com/pledbrook/lazybones).

```bash
lazybones create java-basic properties-apache-commons
```

Previous command will create this structure

```bash
<proj>
      |
      +- src
          |
          +- main
          |     |
          |     +- java
          |
          +- test
          |   |
          |   +- java
```

Edit your `build.gradle` file to create a make a Jar task and add Apache commons dependencies.

```groovy
apply plugin: "java"
apply plugin: "application"

version = '0.0.1'

task buildJar(type: Jar) {
  manifest {
    attributes 'Implementation-Title': 'Reading properties files with Apache Configuration',
    'Implementation-Version': version,
    'Main-Class': 'example.ConfigurationLauncher'
  }
  baseName = project.name + '-all'
  from { configurations.compile.collect { it.isDirectory() ? it : zipTree(it) } }
  with jar
}

repositories {
  mavenCentral()
}

dependencies {
  compile 'org.apache.commons:commons-configuration2:2.0'
  compile 'commons-beanutils:commons-beanutils:1.9.3'
}
```

Then define Java classes under `project-dir/src/main/java/example/`

The easiest way to read configuration is via the `Configurations` helper class. This class offers a bunch of convenience methods for creating configuration objects from different sources. For reading a properties file the code looks as follows:

**ConfigurationLauncher.java**

```java
package example;

import java.io.File;

import org.apache.commons.configuration2.Configuration;
import org.apache.commons.configuration2.builder.fluent.Configurations;
import org.apache.commons.configuration2.ex.ConfigurationException;

public class ConfigurationLauncher {

  private void readProperties(){
    Configurations configs = new Configurations();
    try{
      Configuration config = configs.properties(new File("configuration.properties"));
      String username = config.getString("username");
      String password = config.getString("password");
      System.out.println("{username:" + username + ",password:"+ password + "}");
    }
    catch (ConfigurationException cex) {
      System.out.println("Error: " + cex.getMessage());
    }
  }

  public static void main(String[] args){
    new ConfigurationLauncher().readProperties();
  }

}
```

Configuration information is frequently stored in properties files. Consider the following simple file that defines some properties related to storing user's credentials.

```
username=josdem
password=12345678
```

That's it `configs.properties` will read our `configuration.properties` file from the class path.

To build the project type:

```bash
gradle buildJar
```

To run the project go to `<Project>/build/libs` and type:

```bash
java -jar properties-apache-commons-all-0.0.1.jar
```

To download the code:

```bash
git clone https://github.com/josdem/java-topics.git
cd properties-apache-commons
```


[Return to the main article](/techtalk/java)
