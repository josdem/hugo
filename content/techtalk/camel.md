+++
date = "2016-03-05T07:18:24-06:00"
draft = true
title = "Camel"

+++

Camel is an integration framework aims to simplify this task using a routing engine builder. Camel is a mature open source started in 2007 under the liberal Apache 2 licence, and it has a strong community.

## Camel Characteristics

* Routing and meditation engine
* Domain Specific Language (DSL)
* Payload-agnostic router
* POJO model
* Automatic type converters
* Test kit
* Enterprise Integration patters
* Modular and pluggable technology
* Easy configuration
* Lightweight core
* Vibrant community

## Hello World with Camel

Let's route files, suppose you need to read files from one directory, process in some way and write the result in another directory. Please consider the following example:

```groovy
package com.josdem.hello

import org.apache.camel.CamelContext
import org.apache.camel.impl.DefaultCamelContext
import org.apache.camel.builder.RouteBuilder

class FileCopierWithCamel {
  CamelContext context
  String source = "src/main/resources/source"
  String destination = "src/main/resources/destination"

  FileCopierWithCamel(){
    context = new DefaultCamelContext()
    context.addRoutes(new RouteBuilder(){
      void configure(){
        from("file:${source}?noop=true").to("file:${destination}")
      }
    })
  }

  def start(){
    context.start()
  }

  def stop(){
    context.stop()
  }
}

```

Routes in Camel are defined in such a way that they flow when read. This route can be read like this: consume message from file location in `src/main/resources/source` with the noop option set, and send to file location in `src/main/resource/destination`. The noop option tells Camel to leave the source file as is.

The following Spock test executes our copy file:

```groovy
package com.josdem.hello

import spock.lang.Specification

class FileCopierWithCamelSpec extends Specification {

  FileCopierWithCamel copier = new FileCopierWithCamel()

  void "should copy a file using camel"(){
  given:"An camel context intialization "
    copier.start()
  when:"We wait for camel copy a file and stop context"
    Thread.sleep(5000)
    copier.stop()
  then:"We expect file copied"
    File destination = new File("src/main/resources/destination/message.txt")
    destination.text.contains('Hello Camel')
  }

}
```

To download the project and execute the example:

```bash
git clone git@github.com:josdem/camel-workshop.git
cd hello-camel
gradle -Dtest.single=FileCopierWithCamelSpec test
```

[Return to the main article](/techtalk/techtalks)

