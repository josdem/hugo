+++
title =  "Spring Webflux JAXB"
description = "This technical post shows how to serialize and deserialize with JAXB and Spring Webflux"
date = "2019-11-09T09:30:47-05:00"
tags = ["josdem", "techtalks","programming","technology","Spring Webflux", "JAXB"]
categories = ["techtalk", "code","java", "spring webflux"]
+++

In this technical post we will take a look at how to produce [XML](https://en.wikipedia.org/wiki/XML) using [JAXB](https://eclipse-ee4j.github.io/jaxb-ri/) Jakarta EE implementation. Java Architecture for XML Binding (JAXB) provides an API and tools that automate the mapping between XML documents and Java objects. **NOTE:** If you need to know what tools you need to have installed in your computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot). Let's start creating a new Spring Webflux with Lombok as dependency:

```bash
 spring init --dependencies=webflux,lombok --build=gradle --language=java spring-webflux-jaxb
```

Here is the complete `build.gradle` file generated:

```groovy
plugins {
	id 'org.springframework.boot' version '2.2.1.RELEASE'
	id 'io.spring.dependency-management' version '1.0.8.RELEASE'
	id 'java'
}

group = 'com.jos.dem.spring.webflux.jaxb'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
	compileOnly {
		extendsFrom annotationProcessor
	}
}

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-webflux'
	compileOnly 'org.projectlombok:lombok'
	annotationProcessor 'org.projectlombok:lombok'
	testImplementation('org.springframework.boot:spring-boot-starter-test') {
		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
	testImplementation 'io.projectreactor:reactor-test'
}

test {
	useJUnitPlatform()
}
```

Then add JAXB api and runtime dependencies:

```groovy
implementation "javax.xml.bind:jaxb-api"
implementation "org.glassfish.jaxb:jaxb-runtime"
```

Let's start adding a controller to retreive a XML

```java
package com.jos.dem.spring.webflux.jaxb.controller;

import static org.springframework.http.MediaType.APPLICATION_XML_VALUE;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import reactor.core.publisher.Mono;

import com.jos.dem.spring.webflux.jaxb.model.Person;
import com.jos.dem.spring.webflux.jaxb.repository.PersonRepository;

@RestController
public class PersonController {

  @Autowired
  private PersonRepository personRepository;

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @GetMapping(value = "/", produces = APPLICATION_XML_VALUE)
  public Mono<Person> index() {
    log.info("Getting Person");
    return personRepository.findOne("josdem");
  }

}
```

That's it, adding `APPLICATION_XML_VALUE` as [MediaType](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/MediaType.html) to the `@GetMapping` annotation will do the magic :). Here is our `Person` model:

```java
package com.jos.dem.spring.webflux.jaxb.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.xml.bind.annotation.XmlRootElement;
import javax.xml.bind.annotation.XmlAttribute;
import javax.xml.bind.annotation.XmlElement;

@Getter
@AllArgsConstructor
@NoArgsConstructor
@XmlRootElement
public class Person {
  @XmlAttribute
  private String nickname;
  @XmlAttribute
  private String firstName;
  @XmlAttribute
  private String lastName;
  @XmlAttribute
  private String address;
  @XmlElement
  private Device device;
}
```

Where:

* `@XmlRootElement` annotation associate a root element with our `Person` class.
* `@XmlAttribute` Will map to a XML field attribute.
* `@XmlElement` Will map to a XML element.

Where is our `Device` model:

```java
package com.jos.dem.spring.webflux.jaxb.model;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.xml.bind.annotation.XmlRootElement;
import javax.xml.bind.annotation.XmlAttribute;

@Getter
@AllArgsConstructor
@NoArgsConstructor
@XmlRootElement
public class Device {
  @XmlAttribute
  private String name;
  @XmlAttribute
  private String os;
  @XmlAttribute
  private String model;
}
```

Finally, let's test our controller with [WebTestClient](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/reactive/server/WebTestClient.html)

```java
package com.jos.dem.spring.webflux.jaxb;

import com.jos.dem.spring.webflux.jaxb.model.Person;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.reactive.server.WebTestClient;

import static org.hamcrest.Matchers.equalTo;
import static org.springframework.http.MediaType.APPLICATION_XML_VALUE;

@SpringBootTest(classes = DemoApplication.class,
				webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class DemoApplicationTests {

	@Autowired
	private WebTestClient webClient;

	@Test
	void shouldGetPerson() {
		webClient.get().uri("/")
						.exchange()
						.expectStatus().isOk()
						.expectHeader().contentType(APPLICATION_XML_VALUE)
						.expectBody(Person.class)
						.value(person -> person.getFirstName(), equalTo("Jose"))
						.value(person -> person.getLastName(), equalTo("Morales"))
						.value(person -> person.getAddress(), equalTo("30 Frank Lloyd, Ann Arbor MI 48105"))
						.value(person -> person.getDevice().getName(), equalTo("Pixel 3"))
						.value(person -> person.getDevice().getOs(), equalTo("Android"))
						.value(person -> person.getDevice().getModel(), equalTo("9 Pie"));
	}

}
```

If you want to test this project from command line

```bash
curl -v http://localhost:8080 \
-H 'Content-Type: application/xml' localhost:8080
```

You should see an output similar to this:

```bash
* Connected to localhost (127.0.0.1) port 8080 (#0)
> GET / HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.58.0
> Accept: */*
> Content-Type: application/xml
>
< HTTP/1.1 200 OK
< Content-Type: application/xml
< Content-Length: 222
<
* Connection #0 to host localhost left intact
<?xml version="1.0" encoding="UTF-8" standalone="yes"?><person nickname="josdem" firstName="Jose" lastName="Morales" address="30 Frank Lloyd, Ann Arbor MI 48105"><device name="Pixel 3" os="Android" model="9 Pie"/></person>
```

To browse the project go [here](https://github.com/josdem/spring-webflux-jaxb), to download the project:

```bash
git clone git@github.com:josdem/spring-webflux-jaxb.git
```

To run the project:

```bash
gradle bootRun
```

To test the project:

```bash
gradle test
```

[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
