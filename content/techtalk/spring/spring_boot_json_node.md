+++
title =  "Spring Boot JsonNode"
description = "Marshall and unmarshall using Jackson, JsonNode and Spring Boot"
date = "2019-03-10T10:02:22-04:00"
tags = ["josdem", "techtalks","programming","technology","spring boot","json node","jackson"]
categories = ["techtalk", "code","spring boot"]
+++

In computer science, marshalling is the process of transforming the representation of an object to a data format suitable for storage or transmission. In this technical post we will describe how we can marshall and unmarshall using [Jackson](https://en.wikipedia.org/wiki/Jackson_(API)), Json Node and Spring Boot. If you want to know more about how to create Spring Webflux please go to my previous post getting started with Spring Webflux [here](/techtalk/spring/spring_webflux_basics). First let's create a new Spring Boot Webflux project:

```bash
spring init --dependencies=webflux,lombok --build=gradle --language=java spring-boot-json-node
```

This is the `build.gradle` file generated:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.3.RELEASE'
  id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.json.node'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

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
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
  testImplementation 'io.projectreactor:reactor-test'
}
```

Now let's add Junit5  dependency:

```groovy
testImplementation "org.junit.jupiter:junit-jupiter-api:5.4.0"
testRuntime "org.junit.jupiter:junit-jupiter-engine:5.4.0"
```

**From Json to JsonNode**

Let's validate Json to JsonNode transformation, in order to parse a Json string we only need an `ObjectMapper`

```java
ObjectMapper mapper = new ObjectMapper();
String json = "{\"id\":1196,\"nickname\":\"josdem\",\"email\":\"joseluis.delacruz@gmail.com\"}";
JsonNode node = mapper.readTree(json);
```

Please, consider this test case to validate our transformation to JsonNode.

```java
@Test
@DisplayName("Validate Json to Json Node transformation")
void shouldGetJsonNodeFromJson() throws Exception {
  log.info("Running: Validate json to json node transformation at {}", new Date());

  assertAll("node",
    () -> assertEquals(1196, node.get("id").intValue(), "Should get id"),
    () -> assertEquals("josdem", node.get("nickname").textValue(), "Should get nickname"),
    () -> assertEquals("email", node.get("email").textValue(), "should get email")
  );

}
```

**JsonNode to Bean**

Next, let's validate a JsonNode to [POJO](https://en.wikipedia.org/wiki/Plain_old_Java_object) transformation

```java
package com.jos.dem.springboot.json.node.model;

import lombok.Setter;
import lombok.Getter;
import lombok.AllArgsConstructor;

@Setter
@Getter
@AllArgsConstructor
public class Person {
  private Integer id;
  private String nickname;
  private String email;
}
```

Here is our test case

```java
@Test
@DisplayName("Validate Json Node to Person transformation")
void shouldGetPersonFromJsonNode() throws Exception {
  log.info("Running: Validate json to json node transformation at {}", new Date());

  Person person = mapper.treeToValue(node, Person.class);

  assertAll("person",
    () -> assertEquals(1196, person.getId(), "Should get id"),
    () -> assertEquals("josdem", person.getNickname(), "Should get nickname"),
    () -> assertEquals("email", person.getEmail(), "should get email")
  );

}
```

**Bean to JsonNode**


Now, let's validate a [POJO](https://en.wikipedia.org/wiki/Plain_old_Java_object) to JsonNode transformation

```java
@Test
@DisplayName("Validate Person to Json Node transformation")
void shouldGetJsonNodeFromPerson() throws Exception {
  log.info("Running: Validate person to json node transformation at {}", new Date());
  Person person = new Person(1196, "josdem","joseluis.delacruz@gmail.com");
  JsonNode node = mapper.valueToTree(person);

  assertAll("person",
    () -> assertEquals(1196, node.get("id").intValue(), "Should get id"),
    () -> assertEquals("josdem", node.get("nickname").textValue(), "Should get nickname"),
    () -> assertEquals("email", node.get("email").textValue(), "should get email")
  );

}
```

**Parameters to JsonNode**

Finally let's create a JsonNode with just arguments

```java
@Test
@DisplayName("Validate Arguments to Json Node transformation")
void shouldGetJsonNodeFromArguments() throws Exception {
  log.info("Running: Validate arguments to json node transformation at {}", new Date());
  Integer id = 1196;
  String nickname = "josdem";
  String email = "joseluis.delacruz@gmail.com";

  JsonNode node = mapper.createObjectNode();
  ((ObjectNode) node).put("id", 2016);
  ((ObjectNode) node).put("name", "baeldung.com");

  assertAll("person",
    () -> assertEquals(1196, node.get("id").intValue(), "Should get id"),
    () -> assertEquals("josdem", node.get("nickname").textValue(), "Should get nickname"),
    () -> assertEquals("email", node.get("email").textValue(), "should get email")
  );

}
```

Here is our complete test case

```java
package com.jos.dem.springboot.json.node;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.Date;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.node.ObjectNode;

import com.jos.dem.springboot.json.node.model.Person;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class JsonNodeTest {

  private JsonNode node;
  private ObjectMapper mapper = new ObjectMapper();
  private Logger log = LoggerFactory.getLogger(this.getClass());

  @BeforeEach
  void init() throws Exception {
    log.info("Getting Json Node from Json");
    String json = "{\"id\":1196,\"nickname\":\"josdem\",\"email\":\"joseluis.delacruz@gmail.com\"}";
    node = mapper.readTree(json);
  }


  @Test
  @DisplayName("Validate Json to Json Node transformation")
  void shouldGetJsonNodeFromJson() throws Exception {
    log.info("Running: Validate json to json node transformation at {}", new Date());

    assertAll("node",
      () -> assertEquals(1196, node.get("id").intValue(), "Should get id"),
      () -> assertEquals("josdem", node.get("nickname").textValue(), "Should get nickname"),
      () -> assertEquals("email", node.get("email").textValue(), "should get email")
    );

  }

  @Test
  @DisplayName("Validate JsonNode to Person transformation")
  void shouldGetPersonFromJsonNode() throws Exception {
    log.info("Running: Validate json to json node transformation at {}", new Date());

    Person person = mapper.treeToValue(node, Person.class);

    assertAll("person",
      () -> assertEquals(1196, person.getId(), "Should get id"),
      () -> assertEquals("josdem", person.getNickname(), "Should get nickname"),
      () -> assertEquals("email", person.getEmail(), "should get email")
    );

  }

  @Test
  @DisplayName("Validate Person to JsonNode transformation")
  void shouldGetJsonNodeFromPerson() throws Exception {
    log.info("Running: Validate person to json node transformation at {}", new Date());
    Person person = new Person(1196, "josdem","joseluis.delacruz@gmail.com");
    JsonNode node = mapper.valueToTree(person);

    assertAll("person",
      () -> assertEquals(1196, node.get("id").intValue(), "Should get id"),
      () -> assertEquals("josdem", node.get("nickname").textValue(), "Should get nickname"),
      () -> assertEquals("email", node.get("email").textValue(), "should get email")
    );

  }

  @Test
  @DisplayName("Validate Arguments to Json Node transformation")
  void shouldGetJsonNodeFromArguments() throws Exception {
    log.info("Running: Validate arguments to json node transformation at {}", new Date());
    Integer id = 1196;
    String nickname = "josdem";
    String email = "joseluis.delacruz@gmail.com";

    JsonNode node = mapper.createObjectNode();
    ((ObjectNode) node).put("id", 2016);
    ((ObjectNode) node).put("name", "baeldung.com");

    assertAll("person",
      () -> assertEquals(1196, node.get("id").intValue(), "Should get id"),
      () -> assertEquals("josdem", node.get("nickname").textValue(), "Should get nickname"),
      () -> assertEquals("email", node.get("email").textValue(), "should get email")
    );

  }

  @AfterEach
  void finish() throws Exception {
    log.info("Test execution finished");
  }

}
```

**Dealing with complex Json structures**

Let's consider this Json structure

```json
{
  "batchId": "fbe07c89-ffa7-4c86-9832-5f75cf765737",
  "eventType": "EmployeeClockIn",
  "publishedAt": "2019-03-09T07:36:43-05:00",
  "messages" : [{
  	"messageId" : "4aeaa175-e46d-42eb-83d3-cd02865d4863",
  	"eventAt": "2019-03-09T07:36:43-05:00",
  	"data": {
  		"firstname": "Jose",
  		"lastname": "Morales",
  		"nickname": "josdem",
  		"employeeNumber": 1196,
  		"email": "joseluis.delacruz@gmail.com",
  		"clockInDateTime": "2019-03-09T07:36:43-05:00"
  	}
  }]
}
```

This could be our bean representation: `Event`

```java
package com.jos.dem.springboot.json.node.model;

import java.time.OffsetDateTime;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class Event {
  private String batchId;
  private String eventType;
  private OffsetDateTime publishedAt;
  private Message[] messages;
}
```

`Message`

```java
package com.jos.dem.springboot.json.node.model;

import java.time.OffsetDateTime;

import lombok.Getter;
import lombok.Setter;

import com.fasterxml.jackson.databind.JsonNode;

@Getter
@Setter
public class Message {
  private String messageId;
  private OffsetDateTime eventAt;
  private JsonNode data;
}
```

So to unmarshall this complex Json structure the best strategy is to create a service to delegate that responsibility

```java
package com.jos.dem.springboot.json.node.service;

import java.io.File;
import java.io.IOException;

import com.jos.dem.springboot.json.node.model.Event;

public interface UnmarshallerService {

  Event read(File jsonFile) throws IOException;

}
```

Here is our service implementation

```java
package com.jos.dem.springboot.json.node.service.impl;

import java.io.File;
import java.io.InputStream;
import java.io.FileInputStream;
import java.io.IOException;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

import org.springframework.stereotype.Service;

import com.jos.dem.springboot.json.node.model.Event;
import com.jos.dem.springboot.json.node.service.UnmarshallerService;

@Service
public class UnmarshallerServiceImpl implements UnmarshallerService {

  private ObjectMapper mapper = new ObjectMapper();

  public Event read(File jsonFile) throws IOException {
    InputStream inputStream = new FileInputStream(jsonFile);
    JsonNode jsonNode = mapper.readTree(inputStream);
    return mapper.treeToValue(jsonNode, Event.class);
  }

}
```

And here is our complete test case to unmarshall it

```java
package com.jos.dem.springboot.json.node;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.Date;
import java.time.OffsetDateTime;

import com.fasterxml.jackson.databind.JsonNode;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.extension.ExtendWith;

import java.io.File;
import java.io.InputStream;
import java.io.FileInputStream;

import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit.jupiter.SpringExtension;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.springboot.json.node.model.Event;
import com.jos.dem.springboot.json.node.model.Message;
import com.jos.dem.springboot.json.node.service.UnmarshallerService;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@ExtendWith(SpringExtension.class)
@SpringBootTest
public class UnmarshallerServiceTest {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Autowired
  private UnmarshallerService service;

  private Event event;
  private Message message;

  @BeforeEach
  void init() throws Exception {
    log.info("Getting Event from ClockIn json file");
    File jsonFile = new File("src/main/resources/ClockIn.json");
    event = service.read(jsonFile);
  }

  @Test
  @DisplayName("Validate Event values from ClockIn Json file")
  void shouldGetEventFromClockInFile() throws Exception {
    log.info("Running: Validate Event values from ClockIn Json file at {}", new Date());

    assertAll("event",
      () -> assertEquals("fbe07c89-ffa7-4c86-9832-5f75cf765737", event.getBatchId(), "Should get batch id"),
      () -> assertEquals("EmployeeClockIn", event.getEventType(), "Should get event type"),
      () -> assertEquals(OffsetDateTime.parse("2019-03-09T07:36:43-05:00"), event.getPublishedAt(), "Should get published at time")
    );

  }

  @Test
  @DisplayName("Validate Message values from Event")
  void shouldGetMessageFromEvent() throws Exception {
    log.info("Running: Validate Message values from event at {}", new Date());

    Message[] messages = event.getMessages();
    message = messages[0];

    assertEquals("1", messages.length, "Should contain one message");

    assertAll("message",
      () -> assertEquals("4aeaa175-e46d-42eb-83d3-cd02865d4863", message.getMessageId(), "Should get message id"),
      () -> assertEquals(OffsetDateTime.parse("2019-03-09T07:36:43-05:00"), event.getPublishedAt(), "Should get published at time")
    );

  }

  @Test
  @DisplayName("Validate Data values from message")
  void shouldGetDataFromMessage() throws Exception {
    log.info("Running: Validate Data values from message at {}", new Date());

    JsonNode data = message.getData();
    assertAll("data",
      () -> assertEquals("Jose", data.get("firstname").textValue(), "Should get Firstname"),
      () -> assertEquals("Morales", data.get("lastname").textValue(), "Should get Lastname"),
      () -> assertEquals("josdem", data.get("nickname").textValue(), "Should get Nickname"),
      () -> assertEquals(1196, data.get("employeeNumber").intValue(), "Should get Employee Number"),
      () -> assertEquals("joseluis.delacruz@gmail.com", data.get("email").textValue(), "Should get Email"),
      () -> assertEquals("2019-03-09T07:36:43-05:00", data.get("clockInDateTime"), "Should get clockIn time")
    );

  }

  @AfterEach
  void finish() throws Exception {
    log.info("Test execution finished");
  }

}
```

To browse the project go [here](https://github.com/josdem/spring-boot-json-node), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-json-node.git
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
