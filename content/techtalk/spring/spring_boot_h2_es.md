+++
title =  "Spring Boot y H2"
description = "Usando Spring Boot y Base de Datos H2"
date = "2019-09-15T18:46:07-05:00"
tags = ["josdem", "techtalks","programming","technology","H2", "spring boot"]
categories = ["techtalk", "code", "spring boot"]
+++

En este post técnico veremos como integrar una base de datos en memoria [H2](https://www.h2database.com/html/main.html) y una aplicación Spring Webflux. **Nota:** Si quieres saber que herramientas necesitas tener instaladas en tu computadora para poder familiarizarte con Spring Boot por favor visita mi previo post: [Spring Boot](/techtalk/spring/spring_boot). Entonces ejecuta este comando desde la terminal.

```bash
spring init --dependencies=webflux,lombok,jpa,h2 --language=java --build=gradle spring-boot-h2
```

Aquí está el `build.gradle` generado:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.4.RELEASE'
  id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.h2'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
  mavenCentral()
}

dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
  implementation 'org.springframework.boot:spring-boot-starter-webflux'
  runtimeOnly 'com.h2database:h2'
  compileOnly('org.projectlombok:lombok')
  annotationProcessor('org.projectlombok:lombok')
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
  testImplementation 'io.projectreactor:reactor-test'
}
```

Vamos a empezar creando un modelo persona, así podemos almacenar esa entidad en nuestra base de datos H2.

```java
package com.jos.dem.springboot.h2.model;

import javax.persistence.Id;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;

import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.Getter;
import lombok.Setter;

@Entity
@NoArgsConstructor
@AllArgsConstructor
@Data
public class Person {

	@Id
	@GeneratedValue
	private Long id;
	private String nickname;
	private String email;

}
```

Aquí está nuestro `PersonRepository`, que extendiendo de `JpaRepository` obtenemos algunos métodos `CRUD` que nos permite salvar, buscar personas y demás, además permitirá al repositorio Spring Data JPA escanear en el classpath por esta interface y crear los beans adecuados.

```java
package com.jos.dem.springboot.h2.repository;

import java.util.List;

import org.springframework.data.jpa.repository.JpaRepository;
import com.jos.dem.springboot.h2.model.Person;

public interface PersonRepository extends JpaRepository<Person, Long>{

  Person save(Person person);
  List<Person> findAll();

}
```

El siguiente paso es usar `CommandLineRunner` para crear un registro persona en nuestra tabla `person`. El `CommandLineRunner` es una interfaz call back en Spring Boot, cuando nuestra aplcación arranca llamará al método start y le pasará los argumentos usando el método interno `run()`.

```java
package com.jos.dem.springboot.h2;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.autoconfigure.domain.EntityScan;
import org.springframework.context.annotation.Bean;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

import com.jos.dem.springboot.h2.model.Person;
import com.jos.dem.springboot.h2.repository.PersonRepository;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@SpringBootApplication
@EntityScan("com.jos.dem.springboot.h2.model")
@EnableJpaRepositories("com.jos.dem.springboot.h2.repository")
public class H2Application {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  public static void main(String[] args) {
    SpringApplication.run(H2Application.class, args);
  }

  @Bean
  CommandLineRunner start(PersonRepository personRepository){
    return args -> {
      Person person  = new Person(1L, "josdem", "joseluis.delacruz@gmail.com");
      personRepository.save(person);
      personRepository.findAll().forEach(p -> log.info("person: {}", p));
    };
  }

}
```

H2 provee una interfaz web llamada consola H2. Vamos a habilitarla usando nuestro `application.properties`.

```properties
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
spring.datasource.url=jdbc:h2:mem:persondb
spring.datasource.username=sa
spring.datasource.password=
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.platform=h2
```

El paso final es agregar la dependencia `spring-boot-starter-web` así podemos hacer funcionar la consola H2.

```groovy
implementation 'org.springframework.boot:spring-boot-starter-web'
```

Ahora, si ejecutas nuestra aplicación, deberías ver una salida parecida a esta:

```bash
2019-05-02 09:14:42.378  INFO 8960 --- [main] com.jos.dem.springboot.h2.H2Application  : Started H2Application in 4.944 seconds (JVM running for 5.357)
2019-05-02 09:14:42.492  INFO 8960 --- [main] o.h.h.i.QueryTranslatorFactoryInitiator  : HHH000397: Using ASTQueryTranslatorFactory
2019-05-02 09:14:42.609  INFO 8960 --- [main] ication$$EnhancerBySpringCGLIB$$de299e81 : person: Person(id=1, nickname=josdem, email=joseluis.delacruz@gmail.com)
```

Y acceder a la consola H2 en esta URL: [http://localhost:8080/h2-console](http://localhost:8080/h2-console)

Para correr el proyecto:

```bash
gradle bootRun
```

Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/spring-boot-h2), para descargar el proyecto:

```bash
git clone git@github.com:josdem/spring-boot-h2.git
```

[Regresar al artículo principal](/techtalk/spring#Spring_Boot_Reactive_ES)
