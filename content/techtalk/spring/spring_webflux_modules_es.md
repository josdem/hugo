+++
title =  "Spring Webflux Multi-Módulo"
description = "Wow to build a multi-module project using Gradle and Maven with Spring Webflux"
date = "2018-12-16T10:48:18-05:00"
tags = ["josdem", "techtalks","programming","technology","spring boot"]
categories = ["techtalk", "code", "spring boot"]
+++

En este post técnico veremos como construir un proyecto multi-modulo usando Gradle y Maven con Spring Webflux. **NOTA:** Si quieres saber que necesitas tener instalado en tu computadora para crear fácilmente un proyecto de Spring boot, por favor visita mi previo post: [Spring Boot](/techtalk/spring_boot). Vamos a empezar creando un  nuevo proyecto de Spring Boot con Webflux como dependencia:

```bash
spring init --dependencies=webflux --build=gradle --language=java library
```

Aquí está el `build.gradle` generado incluyendo los closures del jar y dependencyManagement.

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.6.RELEASE' apply false
  id 'io.spring.dependency-management' version '1.0.6.RELEASE'
  id 'java'
}

jar {
  baseName = 'spring-boot-module-library'
  version = '0.0.1-SNAPSHOT'
}

group = 'com.jos.dem.springboot.module.library'
sourceCompatibility = 11

repositories {
  mavenCentral()
}


dependencies {
  implementation('org.springframework.boot:spring-boot-starter-webflux')
  testImplementation('org.springframework.boot:spring-boot-starter-test')
  testImplementation('io.projectreactor:reactor-test')
}

dependencyManagement {
  imports{
    mavenBom org.springframework.boot.gradle.plugin.SpringBootPlugin.BOM_COORDINATES
  }
}
```

Desde que estamos creando una librería, nosotros queremos que sea usado el manejo de dependencias en este proyecto pero sin aplicar el plugin de Spring Boot. La clase `SpringBootPlugin` provee un `BOM_COORDINATES`, una desventaja de este método es que forzamos a especificar la version del plugin de manejo de depencias. Vamos ahora a crear un servicio, de esta manera lo podemos usar en nuestro módulo de aplicación.

```java
package com.jos.dem.springboot.module.library.service;

import reactor.core.publisher.Mono;

import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Value;

@Service
public class MessageService {

  @Value("${message}")
  private String message;

  public Mono<String> getMessage(){
    return Mono.just(message);
  }

}
```

Ahora, vamnos a crear un proyecto para la aplicación, desde la raíz del `$PROJECT_HOME:`

```bash
spring init --dependencies=webflux --build=gradle --language=java application
```

Aquí está el `build.gradle` generado:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.6.RELEASE'
  id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.module'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 11

repositories {
  mavenCentral()
}

dependencies {
  implementation('org.springframework.boot:spring-boot-starter-webflux')
  implementation project(':library')
  testImplementation('org.springframework.boot:spring-boot-starter-test')
  testImplementation('io.projectreactor:reactor-test')
}
```

Así es, estamos importando nuestra librería como un proyecto dependencia usando la propiedad implementation de Gradle. Ahora es el momento de crear un controlador y los archivos de la aplicación.

```java
package com.jos.dem.springboot.module.controller;

import reactor.core.publisher.Mono;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.springboot.module.library.service.MessageService;

@RestController
public class ModuleController {

  @Autowired
  private MessageService messageService;

  @GetMapping("/")
  public Mono<String> index(){
    return messageService.getMessage();
  }

}
```

Aplicación de Spring Boot:

```java
package com.jos.dem.springboot.module;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ModuleApplication {

  public static void main(String[] args) {
    SpringApplication.run(ModuleApplication.class, args);
  }

}
```

Nosotros necesitaremos proveer un mensaje para el servicio in la librería vía `$PROJECT_HOME/application/src/main/resources/application.properties`.

```properties
message=Hello World!
```

Finalmente, agreguemos un `settings.gradle` al directorio raíz para epecificar los módulos.

```bash
rootProject.name = 'spring-boot-module'

include 'library'
include 'application'
```

Para ejecutar el proyecto se construye la librería primero, entonces se ejecuta la aplicación desde `$PROJECT_HOME`:

```bash
gradle build :application:bootRun
```

**Using Maven**

Tú puedes hacer lo mismo usando Maven, la única diferencia es que tienes que específicar el parámetro `--build=maven` en el comando `spring init`, esta es archivo pom generado de la `library`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem.springboot.module</groupId>
  <artifactId>library</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.4.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>11</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

</project>
```

Aquí está el archivo pom de la aplicación.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem.springboot.module</groupId>
  <artifactId>application</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.4.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>11</java.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
      <groupId>com.jos.dem.springboot.module</groupId>
      <artifactId>library</artifactId>
      <version>0.0.1-SNAPSHOT</version>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>

</project>
```

Y aquí está nuestro `pom.xml` en el directorio `$PROJECT_HOME`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem.springboot.module</groupId>
  <artifactId>spring-boot-module</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>pom</packaging>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.4.RELEASE</version>
    <relativePath /> <!-- lookup parent from repository -->
  </parent>

  <modules>
    <module>library</module>
    <module>application</module>
  </modules>

</project>
```

Se construye la librería primero y entonces se corre la aplicación desde `$PROJECT_HOME`:

```bash
mvn install spring-boot:run -pl application
```

Para ver los resultados por favor abre esta URL en tu browser:

```bash
 http://localhost:8080/
```

Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/spring-boot-modules), para descargar el proyecto:

```bash
git clone git@github.com:josdem/spring-boot-modules.git
```

[Regresar al artículo principal](/techtalk/spring#Spring_Boot_Reactive_ES)
