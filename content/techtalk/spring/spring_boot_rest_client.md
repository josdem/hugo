+++
date = "2017-09-15T08:06:41-05:00"
title = "Spring Boot Rest Client"
categories = ["techtalk","code"]
tags = ["josdem","techtalks","programming","technology"]

+++

This time I will show you how to consume a rest service using Spring Boot and a Groovy library based in HTTPBuilder [RESTClient](https://github.com/jgritman/httpbuilder/wiki/RESTClient) by jgritman. First you need to add the `RESTClient` dependency, this is our complete `build.gradle`


```groovy
buildscript {
  ext {
    springBootVersion = '1.5.6.RELEASE'
  }
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  }
}

apply plugin: 'groovy'
apply plugin: 'org.springframework.boot'

version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
  mavenCentral()
}


dependencies {
  compile('org.springframework.boot:spring-boot-starter-web')
  compile('org.codehaus.groovy.modules.http-builder:http-builder:0.7.1')
  compile('org.codehaus.groovy:groovy-all')
  testCompile('org.springframework.boot:spring-boot-starter-test')
}
```

In this example we are going to consume a rest service for this project [Jugoterapia](https://github.com/josdem/jugoterapia-spring-boot) Which is an Android application mainly focused in improve your healty based in juice recipes, this project is the server side, it is exposing recipes and beverages as API service.

And we will request this recipe as example: http://jugoterapia.josdem.io/jugoterapia-server/beverage/beverage?beverageId=35

```json
{
"id": 35,
"name": "Jugo nutritivo (Zanahoria)",
"ingredients": "4 Zanahorias,1 Tallo de apío,1 Pera,5 hojas de espinacas",
"recipe": "Lava perfectamente todos los ingrendientes. Pasa la zanahoria por el extractor, el apio, las espinacas y la pera. Mezcla todo perfectamente y bebe de inmediato. La espinaca es una excelente fuente de hierro. Promueve el transporte y depósito de oxí­geno en los tejidos, aumenta la fuerza muscular, ayuda a bajar de peso, favorece el tránsito intestinal, beneficia a mujeres embarazadas y niños debido a su contenido de ácido fólico (vitamina B9), mejora la visión y mantiene la presión arterial balanceada."
}
```

So let's create a service to consume:

```groovy
package com.jos.dem.rest.client.service

interface RestClient {

  def getBeverage()

}
```

`RestServiceImpl`

```groovy
package com.jos.dem.rest.client.service.impl

import org.springframework.stereotype.Service

import groovyx.net.http.RESTClient

import com.jos.dem.rest.client.service.RestClient
import com.jos.dem.rest.client.exception.BusinessException

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@Service
class RestClientImpl implements RestClient {

  Logger log = LoggerFactory.getLogger(this.class)

  def getBeverage(){
    try {
      log.info "Calling Service"
      RESTClient restClient = new RESTClient()
      def result = restClient.get(
        uri : 'http://jugoterapia.josdem.io/jugoterapia-server/beverage/beverage?beverageId=35'
      )
      result.data
    } catch(Exception ex) {
      log.warn "Error ${ex.message}"
      throw new BusinessException(ex.message)
    }
  }

}
```

That's it, we are requesting from the endpoint our beverage as an GET request and if something fails we are throwing an `BusinessException`

```groovy
package com.jos.dem.rest.client.exception

class BusinessException extends RuntimeException {

  BusinessException(String msg){
    super(msg)
  }

  BusinessException(String msg, Throwable cause) {
    super(msg, cause)
  }

}
```

In order to consume our service we will get the bean from the Spring application context so we can call `getBeverage()` method.

```groovy
package com.jos.dem.rest.client

import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.context.ConfigurableApplicationContext

import com.jos.dem.rest.client.model.Beverage
import com.jos.dem.rest.client.service.RestClient

@SpringBootApplication
class RestClientApplication {

  static void main(String[] args) {
    ConfigurableApplicationContext context = SpringApplication.run RestClientApplication, args
    Beverage beverage = context.getBean(RestClient.class).getBeverage()
    println "beverage: ${beverage.dump()}"
  }

}
```


To download the project:

```bash
git clone https://github.com/josdem/spring-boot-rest-client.git
```

To run the project:

```bash
gradle bootRun
```

[Return to the main article](/techtalk/spring)

