+++
title =  "Spring Boot y JsonNode"
description = "Spring Boot y JsonNode en Español"
date = "2019-09-15T10:44:41-05:00"
tags = ["josdem", "techtalks","programming","technology","spring boot", "json node"]
categories = ["techtalk", "code", "spring boot", "spring"]
+++

En ciencias computacionales, marshalling es el proceso de transformar la representación de un objeto a un formato de datos que se pueda almacenar o transmitir. En este post técnico describiremos como podemos hacer marshal y unmarshall usando [Jackson](https://en.wikipedia.org/wiki/Jackson_(API)), JsonNode y Spring Boot. **Nota:** Si quieres saber que herramientas necesitas tener instaladas en tu computadora para poder familiarizarte con Spring Boot por favor visita mi previo post: [Spring Boot](/techtalk/spring/spring_boot). Entonces ejecuta este comando desde la terminal.

```bash
spring init --dependencies=webflux,lombok --build=gradle --language=java spring-boot-json-node
```

Aquí está el `build.gradle` generado:

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

Ahora,  agreguemos la dependencia Junit5:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.3.RELEASE'
  id 'java'
}

apply plugin: 'io.spring.dependency-management'

def junitJupiterVersion = '5.4.0'

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


