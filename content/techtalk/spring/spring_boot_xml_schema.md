+++
title =  "Spring Boot XML Schema"
description = "Spring_boot_xml_schema"
date = "2019-09-02T15:48:23-04:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

[XML](https://en.wikipedia.org/wiki/XML) is still the most sophisticated format for distributed data, when you have complete control between your clients and servers [JSON](https://en.wikipedia.org/wiki/JSON) is the king, but if not, you probably will need to deal with XML, and there is for good reasons:

* XML is a [W3C](https://en.wikipedia.org/wiki/World_Wide_Web_Consortium) standard
* XML can be validated using Schemas
* XML embraces extensibility
* Supports multilingual documents
* Can embed multiple data types

All those reasons are important, but most important is that we can validate any XML using [XSD](https://en.wikipedia.org/wiki/XML_Schema_(W3C)) Schemas, in this technical post we will see how to generate Java classes using XSD eschemas with [JAXB2](https://github.com/highsource/maven-jaxb2-plugin/wiki) and Spring Boot. **NOTE:** If you need to know what tools you need to have installed in your computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot). Let's start creating a new Spring Boot project with Webflux as dependency:

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
    <version>2.1.7.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
  <groupId>com.jos.dem.springboot.xml.schema</groupId>
  <artifactId>spring-boot-xml-schema</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>demo</name>
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
2019-09-02 15:34:00.776 INFO 5619 - [main] Application$$EnhancerBySpringCGLIB$$7416dabf : Person: com.jos.dem.entities.Person@3136695c[firstName=Jose,lastName=Morales,address=30 Frank Llord Dr, Ann Arbor, Michigan]
```

To browse the project go [here](https://github.com/josdem/spring-boot-xml-schema), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-xml-schema.git
```

To run the project:

```bash
mvn spring-boot:run
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
