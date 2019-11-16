+++
title =  "Spring Boot y Esquemas XML"
description = "En este post tecnico veremos como generar clases Java usando esquemas XSD con JAXB2"
date = "2019-09-02T15:48:23-04:00"
tags = ["josdem", "techtalks","programming","technology", "xsd", "jaxb2"]
categories = ["techtalk", "code", "spring boot"]
+++

[XML](https://en.wikipedia.org/wiki/XML) sigue siendo el formato más sofisticado para datos distribuidos, cuando tenemos completo control entre los clientes y servicios [JSON](https://en.wikipedia.org/wiki/JSON) es el rey, pero sino, probablemente tendrás que ocupar XML y es por buenas razones:

* XML es un estándar [W3C](https://en.wikipedia.org/wiki/World_Wide_Web_Consortium)
* XML puede ser validado usando esquemas
* XML fomenta la extensibilidad
* Suporta multilenguaje
* Soporta diferentes tipos de datos

Todas esas razones son imporatntes, pero la más imporante es que puede ser validado usando esquemas [XSD](https://en.wikipedia.org/wiki/XML_Schema_(W3C)), en este post técnico veremos como generar clases Java usando esquemas XSD con [JAXB2](https://github.com/highsource/maven-jaxb2-plugin/wiki) y Spring Boot. **NOTA:** Si quieres saber más acerca de como crear una aplicación Spring Webflux por favor visita mi previo post empezando con Spring Webflux [aquí](/techtalk/spring/spring_webflux_basics). Entonces, crea una aplicación Spring Boot con Webflux como dependencia:

```bash
spring init --dependencies=webflux --build=maven --language=java spring-boot-xml-schema
```

Here is the complete `pox.xml` file generated:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.2.1.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
  <groupId>com.jos.dem.springboot.xml.schema</groupId>
  <artifactId>spring-boot-xml-schema</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>spring-boot-xml-schema</name>
  <description>This project shows how to use XML schemas in a Spring Boot application</description>

  <properties>
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
    <dependency>
      <groupId>io.projectreactor</groupId>
      <artifactId>reactor-test</artifactId>
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

Now, please add JAXB api and Apache Commons Lang3 dependencies to your `pox.xml` file:

```xml
<dependency>
  <groupId>javax.xml.bind</groupId>
  <artifactId>jaxb-api</artifactId>
  <version>2.3.1</version>
</dependency>
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-lang3</artifactId>
  <version>3.9</version>
</dependency>
```

And the JAXB2 Maven Plugin so we can transform our XML schemas in Java classes.

```xml
<plugin>
  <groupId>org.jvnet.jaxb2.maven2</groupId>
  <artifactId>maven-jaxb2-plugin</artifactId>
  <executions>
    <execution>
      <id>generate</id>
      <goals>
        <goal>generate</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

Now, under `src/main/resources` let's add a `Person.xsd` schema

```xml
<?xml version="1.0"?>
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
            elementFormDefault="qualified"
            targetNamespace="urn:com:jos:dem:entities">
    <xs:element name="Person" />
    <xs:complexType name="person">
      <xs:sequence>
        <xs:element name="firstName" type="xs:string" />
        <xs:element name="lastName" type="xs:string" />
        <xs:element name="address" type="xs:string" />
      </xs:sequence>
    </xs:complexType>
</xs:schema>
```

Where:

* `xmlns:xs` Is our W3C standard definition
* `elementFormDefault` Makes qualifying the name spaces of the elements in XML documents
* `targetNamespace` Is is name that better identifies your schema.
* `complexType` Group all Person's attributes
* `sequence` element makes sure our attributes stay in order.

This could be a XML representation

```xml
<?xml version="1.0" encoding="UTF-8"?>
<person xmlns="urn:com:jos:dem:entities"
     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="urn:com:jos:dem:entities
     file:./person.xsd">

  <firstname>Jose</fistname>
  <lastname>Morales</lastname>
  <address>
    30 Frank Llord,
    Ann Arbor,
    Michigan
  </address>
</person>
```

Let's move forward and create a service to get that specific person


```java
package com.jos.dem.springboot.xml.schema.service;

import com.jos.dem.entities.Person;

public interface PersonService {

  Person create();

}
```

This is the implementation

```java
package com.jos.dem.springboot.xml.schema.service;

import com.jos.dem.entities.Person;

import org.springframework.stereotype.Service;

@Service
public class PersonServiceImpl implements PersonService {

  public Person create(){
    Person person =  new Person();
    person.setFirstName("Jose");
    person.setLastName("Morales");
    person.setAddress("30 Frank Llord Dr, Ann Arbor, Michigan");
    return person;
  }

}
```

Now, we can use `CommandLineRunner` to call our service and print out our person entity

```java
package com.jos.dem.springboot.xml.schema;

import org.springframework.context.annotation.Bean;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import org.apache.commons.lang3.builder.ToStringBuilder;

import com.jos.dem.springboot.xml.schema.service.PersonService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@SpringBootApplication
public class DemoApplication {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  public static void main(String[] args) {
    SpringApplication.run(DemoApplication.class, args);
  }

  @Bean
  CommandLineRunner start(PersonService personService){
    return args -> {
      log.info("Person: {}", ToStringBuilder.reflectionToString(personService.create()));
    };
  }

}
```

The `CommandLineRunner` is a call back interface in Spring Boot, when Spring Boot starts will call it and pass in args through a `run()` internal method. Finally, if you execute our Spring Boot application, you should be able to see this output:

```bash
2019-11-16 16:06:54.149  INFO 13286 --- [main] ication$$EnhancerBySpringCGLIB$$5c2c9c7e : Person: com.jos.dem.entities.Person@459cfcca[firstName=Jose,lastName=Morales,address=30 Frank Lloyd, Ann Arbor MI 48105]
```

Finally, let’s test our person service

```java
package com.jos.dem.springboot.xml.schema;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import com.jos.dem.entities.Person;
import com.jos.dem.springboot.xml.schema.service.PersonService;

@SpringBootTest
class DemoApplicationTests {

  @Autowired
  private PersonService personService;

  @Test
  @DisplayName("Should create person")
  void shouldCreatePerson() {
    Person person = personService.create();

    assertAll("person",
        () -> assertEquals("Jose", person.getFirstName(), "should get first name"),
        () -> assertEquals("Morales", person.getLastName(), "should get last name"),
        () -> assertEquals("30 Frank Lloyd, Ann Arbor MI 48105", person.getAddress(), "should get address")
        );
  }

}
```

To browse the project go [here](https://github.com/josdem/spring-boot-xml-schema), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-xml-schema.git
```

To run the project:

```bash
mvn spring-boot:run
```

To test the project:

```bash
mvn test
```


[Regresar al artículo principal](/techtalk/spring#Spring_Boot_Reactive_ES)
