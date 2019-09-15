+++
title =  "Spring Boot y JsonNode"
description = "Spring Boot y JsonNode en Español"
date = "2019-09-15T10:44:41-05:00"
tags = ["josdem", "techtalks","programming","technology","spring boot", "json node"]
categories = ["techtalk", "code", "spring boot", "spring"]
+++

En ciencias computacionales, marshalling es el proceso de transformar la representación de un objeto a un formato de datos que se pueda almacenar o transmitir. En este post técnico describiremos como podemos hacer marshall y unmarshall usando [Jackson](https://en.wikipedia.org/wiki/Jackson_(API)), JsonNode y Spring Boot. **Nota:** Si quieres saber que herramientas necesitas tener instaladas en tu computadora para poder familiarizarte con Spring Boot por favor visita mi previo post: [Spring Boot](/techtalk/spring/spring_boot). Entonces ejecuta este comando desde la terminal.

```bash
spring init --dependencies=webflux,lombok --build=gradle --language=java spring-boot-json-node
```

Aquí está el `build.gradle` generado:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.7.RELEASE'
  id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.json.node'
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
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
  testImplementation 'io.projectreactor:reactor-test'
}
```

Ahora,  agreguemos la dependencia Junit5:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.7.RELEASE'
  id 'java'
}

apply plugin: 'io.spring.dependency-management'

def junitJupiterVersion = '5.4.1'

group = 'com.jos.dem.springboot.json.node'
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
  compileOnly('org.projectlombok:lombok')
  annotationProcessor('org.projectlombok:lombok')
  testImplementation('org.springframework.boot:spring-boot-starter-test'){
    exclude group: 'junit', module: 'junit'
  }
  testImplementation "org.junit.jupiter:junit-jupiter-api:$junitJupiterVersion"
  testRuntime "org.junit.jupiter:junit-jupiter-engine:$junitJupiterVersion"
  testImplementation 'io.projectreactor:reactor-test'
  testCompile 'org.junit.platform:junit-platform-commons:1.4.0'
  testCompile 'org.junit.platform:junit-platform-launcher:1.4.0'
}

test {
  useJUnitPlatform()
}
```

**De Json a JsonNode**

Ahora, hagamos la transformación de Json a JsonNode, para poder parsear un Json necesitaremos un `ObjectMapper`

```java
ObjectMapper mapper = new ObjectMapper();
String json = "{\"id\":1196,\"nickname\":\"josdem\",\"email\":\"joseluis.delacruz@gmail.com\"}";
JsonNode node = mapper.readTree(json);
```

Por favor, considera este test case para validar la transformación a JsonNode.

```java
@Test
@DisplayName("Validate Json to JsonNode transformation")
void shouldGetJsonNodeFromJson() throws Exception {
  log.info("Running: Validate json to json node transformation at {}", new Date());

  assertAll("node",
    () -> assertEquals(1196, node.get("id").intValue(), "Should get id"),
    () -> assertEquals("josdem", node.get("nickname").textValue(), "Should get nickname"),
    () -> assertEquals("joseluis.delacruz@gmail.com", node.get("email").textValue(), "should get email")
  );

}
```

**De JsonNode a Bean**

Después, transformemos de JsonNode a [POJO](https://en.wikipedia.org/wiki/Plain_old_Java_object)

```java
package com.jos.dem.springboot.json.node.model;

import lombok.Setter;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

@Setter
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class Person {
  private Integer id;
  private String nickname;
  private String email;
}
```

Y aquí está el test case:

```java
@Test
@DisplayName("Validate JsonNode to Person transformation")
void shouldGetPersonFromJsonNode() throws Exception {
  log.info("Running: Validate json to json node transformation at {}", new Date());

  Person person = mapper.treeToValue(node, Person.class);

  assertAll("person",
    () -> assertEquals(1196, person.getId(), "Should get id"),
    () -> assertEquals("josdem", person.getNickname(), "Should get nickname"),
    () -> assertEquals("joseluis.delacruz@gmail.com", person.getEmail(), "should get email")
  );

}
```

**De Bean a JsonNode**

Anora, transformemos de [POJO](https://en.wikipedia.org/wiki/Plain_old_Java_object) a JsonNode.

```java
@Test
@DisplayName("Validate Person to JsonNode transformation")
void shouldGetJsonNodeFromPerson() throws Exception {
  log.info("Running: Validate person to json node transformation at {}", new Date());
  Person person = new Person(1196, "josdem","joseluis.delacruz@gmail.com");
  JsonNode node = mapper.valueToTree(person);

  assertAll("person",
    () -> assertEquals(1196, node.get("id").intValue(), "Should get id"),
    () -> assertEquals("josdem", node.get("nickname").textValue(), "Should get nickname"),
    () -> assertEquals("joseluis.delacruz@gmail.com", node.get("email").textValue(), "should get email")
  );

}
```

**Parámetros a JsonNode**

Finalmente, transformemos sólo argumentos a JsonNode

```java
@Test
@DisplayName("Validate Arguments to Json Node transformation")
void shouldGetJsonNodeFromArguments() throws Exception {
  log.info("Running: Validate arguments to json node transformation at {}", new Date());
  Integer id = 1196;
  String nickname = "josdem";
  String email = "joseluis.delacruz@gmail.com";

  JsonNode node = mapper.createObjectNode();
  ((ObjectNode) node).put("id", 1196);
  ((ObjectNode) node).put("nickname", "josdem");
  ((ObjectNode) node).put("email", "joseluis.delacruz@gmail.com");

  assertAll("person",
    () -> assertEquals(1196, node.get("id").intValue(), "Should get id"),
    () -> assertEquals("josdem", node.get("nickname").textValue(), "Should get nickname"),
    () -> assertEquals("joseluis.delacruz@gmail.com", node.get("email").textValue(), "should get email")
  );

}
```

Aquí está el test case completo:

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
  @DisplayName("Validate Json to JsonNode transformation")
  void shouldGetJsonNodeFromJson() throws Exception {
    log.info("Running: Validate json to json node transformation at {}", new Date());

    assertAll("node",
      () -> assertEquals(1196, node.get("id").intValue(), "Should get id"),
      () -> assertEquals("josdem", node.get("nickname").textValue(), "Should get nickname"),
      () -> assertEquals("joseluis.delacruz@gmail.com", node.get("email").textValue(), "should get email")
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
      () -> assertEquals("joseluis.delacruz@gmail.com", person.getEmail(), "should get email")
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
      () -> assertEquals("joseluis.delacruz@gmail.com", node.get("email").textValue(), "should get email")
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
    ((ObjectNode) node).put("id", 1196);
    ((ObjectNode) node).put("nickname", "josdem");
    ((ObjectNode) node).put("email", "joseluis.delacruz@gmail.com");

    assertAll("person",
      () -> assertEquals(1196, node.get("id").intValue(), "Should get id"),
      () -> assertEquals("josdem", node.get("nickname").textValue(), "Should get nickname"),
      () -> assertEquals("joseluis.delacruz@gmail.com", node.get("email").textValue(), "should get email")
    );

  }

  @AfterEach
  void finish() throws Exception {
    log.info("Test execution finished");
  }

}
```

**Trabajando con Estrúcturas Json Complejas**

Consideremos esta estructura Json

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

Este podría una representación en Objeto, `Event`:

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

Este podría ser nuestra representación Objecto `Message`:

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

Así tenemos que para hacer unmarshall de esta estrúctura Json compleja la mejor estrategía es crear un servicio para delegar la responsabilidad.

```java
package com.jos.dem.springboot.json.node.service;

import java.io.File;
import java.io.IOException;

import com.jos.dem.springboot.json.node.model.Event;

public interface UnmarshallerService {

  Event read(File jsonFile) throws IOException;

}
```

Aquí está la implementación de nuestro servicio:

```java
package com.jos.dem.springboot.json.node.service.impl;

import java.io.File;
import java.io.InputStream;
import java.io.FileInputStream;
import java.io.IOException;

import javax.annotation.PostConstruct;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;

import org.springframework.stereotype.Service;

import com.jos.dem.springboot.json.node.model.Event;
import com.jos.dem.springboot.json.node.service.UnmarshallerService;

@Service
public class UnmarshallerServiceImpl implements UnmarshallerService {

  private ObjectMapper mapper = new ObjectMapper();

  @PostConstruct
  public void setup() {
    mapper.registerModule(new JavaTimeModule());
  }

  public Event read(File jsonFile) throws IOException {
    InputStream inputStream = new FileInputStream(jsonFile);
    JsonNode jsonNode = mapper.readTree(inputStream);
    return mapper.treeToValue(jsonNode, Event.class);
  }

}
```

Aquí está nuestro test case completo para validar el unmarshall:

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
  private Message[] messages;

  @BeforeEach
  void init() throws Exception {
    log.info("Getting Event from ClockIn json file");
    File jsonFile = new File("src/main/resources/ClockIn.json");
    event = service.read(jsonFile);
    Message[] messages = event.getMessages();
    message = messages[0];
  }

  @Test
  @DisplayName("Validate Event values from ClockIn Json file")
  void shouldGetEventFromClockInFile() throws Exception {
    log.info("Running: Validate Event values from ClockIn Json file at {}", new Date());

    assertAll("event",
      () -> assertEquals("fbe07c89-ffa7-4c86-9832-5f75cf765737", event.getBatchId(), "Should get batch id"),
      () -> assertEquals("EmployeeClockIn", event.getEventType(), "Should get event type"),
      () -> assertEquals(OffsetDateTime.parse("2019-03-09T12:36:43Z"), event.getPublishedAt(), "Should get published at time")
    );

  }

  @Test
  @DisplayName("Validate Message values from Event")
  void shouldGetMessageFromEvent() throws Exception {
    log.info("Running: Validate Message values from event at {}", new Date());

    assertAll("message",
      () -> assertEquals("4aeaa175-e46d-42eb-83d3-cd02865d4863", message.getMessageId(), "Should get message id"),
      () -> assertEquals(OffsetDateTime.parse("2019-03-09T12:36:43Z"), event.getPublishedAt(), "Should get published at time")
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
      () -> assertEquals("2019-03-09T07:36:43-05:00", data.get("clockInDateTime").textValue(), "Should get clockIn time")
    );

  }

  @AfterEach
  void finish() throws Exception {
    log.info("Test execution finished");
  }

}
```

**Usando Maven**

Tú puedes hacer lo mismo usando Maven, la única diferencia es que tienes que específicar el parámetro `--build=maven` en el comando `spring init`:

```bash
spring init --dependencies=webflux,lombok --build=maven --language=java spring-boot-json-node
```

Este es el `pom.xml` generado con la dependencia Junit5 agregada manualmente:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem.springboot</groupId>
  <artifactId>jsonNode</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>spring-boot-json-node</name>
  <description>This project shows how to work with Jackson JsonNode</description>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.1.7.RELEASE</version>
    <relativePath/>
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <maven.surefire.version>2.22.0</maven.surefire.version>
    <maven-failsafe-plugin.version>2.22.0</maven-failsafe-plugin.version>
    <java.version>11</java.version>
    <junit.jupiter.version>5.4.1</junit.jupiter.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
      <exclusions>
        <exclusion>
          <groupId>junit</groupId>
          <artifactId>junit</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
    <dependency>
      <groupId>io.projectreactor</groupId>
      <artifactId>reactor-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-api</artifactId>
      <version>${junit.jupiter.version}</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter-engine</artifactId>
      <version>${junit.jupiter.version}</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>org.junit.platform</groupId>
      <artifactId>junit-platform-commons</artifactId>
      <version>1.4.0</version>
    </dependency>
    <dependency>
      <groupId>org.junit.platform</groupId>
      <artifactId>junit-platform-launcher</artifactId>
      <version>1.4.0</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-failsafe-plugin</artifactId>
      </plugin>
    </plugins>
  </build>

</project>
```

Para correr el proyecto con Gradle:

```bash
gradle test
```

Para correr el proyecto con Maven:

```bash
mvn test
```

Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/spring-boot-json-node), para descargar el proyecto:

``bash
git clone git@github.com:josdem/spring-boot-json-node.git
```

[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
