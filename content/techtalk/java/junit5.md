+++
title =  "JUnit 5"
description = "Junit5"
date = "2018-05-02T15:27:13-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code", "java"]
+++

Junit 5 is the next generation of Junit, it requires Java 8 since it was born as JUnit Lambda project, that project wanted to take full advantage of the new features from Java 8, especially lambda expressions.

Using Gradle, you need to create a `build.gradle` file with the following structure:

```groovy
def junitJupiterVersion = '5.3.1'

apply plugin: "java"

repositories {
  mavenCentral()
}

dependencies {
  testCompile "org.junit.jupiter:junit-jupiter-api:$junitJupiterVersion"
  testRuntime "org.junit.jupiter:junit-jupiter-engine:$junitJupiterVersion"
}

test {
  useJUnitPlatform()

  reports {
    html.enabled = true
  }
}
```

In dependencies section we are defining Junit Jupiter api, engine version, `jupiter-engine` is only required at runtime. Also `test` task definition we specify Junit platform support, you can get more information [here](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.testing.Test.html), with `html.engine = true` we can generate html reports in a similar way `Spock Framework` does.

If you want to use Maven, please create the following `pom.xml` structure in the root project:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.jos.dem</groupId>
  <artifactId>junit5</artifactId>
  <version>1.0-SNAPSHOT</version>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>${maven.compiler.source}</maven.compiler.target>
    <junit.jupiter.version>5.3.1</junit.jupiter.version>
  </properties>

  <dependencies>
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
  </dependencies>

  <build>
    <plugins>
      <!-- JUnit 5 requires Surefire version 2.22.0 or higher -->
      <plugin>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>2.22.0</version>
      </plugin>
    </plugins>
  </build>

</project>
```

**Assertions**

Assertions have been moved to `org.junit.jupiter.api.Assertions` and have been improved significantly. As mentioned earlier, you can now use lambdas in assertions:

```java
@Test
@DisplayName("Should show how we can use lambdas in a test")
void shouldTestLambdaExpression() {
  assertTrue(Stream.of(1, 2, 3)
    .mapToInt(Integer::intValue)
    .sum() == 6, () -> "Sum should be 6");
}
```

In Junit5 you can group assertions, so when they are executed failures will be reported together.

```java
Person person = new Person("josdem", "joseluis.delacruz@gmail.com");

@Test
@DisplayName("Should assert all person attributes at once")
void shouldAssertAllPersonAttributes() {
  assertAll("person",
    () -> assertEquals("josdem", person.getNickname()),
    () -> assertEquals("joseluis.delacruz@gmail.com", person.getEmail())
  );
}
```

You can write dependent assertions, so if an assertion fails the subsequent code in the same block will be skipped.

```java
@Test
@DisplayName("Should show how works dependent assertions")
void shouldTestDependentAssertions(){
  assertAll("person",
               () -> {
                 String nickname = person.getNickname();
                 assertNotNull(nickname);

                 assertAll("nickname",
                   () -> assertTrue(nickname.startsWith("j")),
                   () -> assertTrue(nickname.endsWith("m"))
                 );
            });
}
```

In Junit 4 we used to test expected exceptions like this: `@Test(expected = IllegalArgumentException.class)` in Junit 5 we write expected exceptions this way:

```java
@Test
@DisplayName("Should throw an exception")
public void shouldThrowNullPointerException() {
  Person person = null;
  assertThrows(NullPointerException.class, () -> person.getNickname());
}
```

Sometimes you want to temporarily disable a test, so you can use `@Disabled` annotation:

```java
@Test
@Disabled("Should not execute this test")
void shouldSkipThisTest() {
  assertTrue(false);
}
```

It is common to use timeout in some specific test cases, using Junit 5 you can write them as follow:

```java
@Test
@DisplayName("Should show how to run a test before timeout")
void timeoutNotExceededWithMethod() {
  String actualGreeting = assertTimeout(ofMinutes(2), AssertionShowTest::greeting);
  assertEquals("Hello, World!", actualGreeting);
}

private static String greeting() {
  return "Hello, World!";
}
```

Here is the complete Junit test case:

```java
package com.jos.dem.junit;

import static java.time.Duration.ofMinutes;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTimeout;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertNotNull;

import java.util.stream.Stream;

import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

class AssertionShowTest {

  private Person person = new Person("josdem", "joseluis.delacruz@gmail.com");

  @Test
  @DisplayName("Should show how we can use lambdas in a test")
  void shouldTestLambdaExpression() {
    assertTrue(Stream.of(1, 2, 3)
      .mapToInt(Integer::intValue)
      .sum() == 6, () -> "Sum should be 6");
  }

  @Test
  @DisplayName("Should assert all person attributes at once")
  void shouldAssertAllPersonAttributes() {
      assertAll("person",
          () -> assertEquals("josdem", person.getNickname()),
          () -> assertEquals("joseluis.delacruz@gmail.com", person.getEmail())
      );
  }

  @Test
  @DisplayName("Should show how works dependent assertions")
  void shouldTestDependentAssertions(){
    assertAll("person",
            () -> {
                String nickname = person.getNickname();
                assertNotNull(nickname);

                assertAll("nickname",
                    () -> assertTrue(nickname.startsWith("j")),
                    () -> assertTrue(nickname.endsWith("m"))
                );
            });
  }

  @Test
  @DisplayName("Should throw an exception")
  public void shouldThrowNullPointerException() {
    Person person = null;
    assertThrows(NullPointerException.class, () -> person.getNickname());
  }

  @Test
  @Disabled("Should not execute this test")
  void shouldSkipThisTest() {
    assertTrue(false);
  }

  @Test
  @DisplayName("Should show how to run a test before timeout")
  void timeoutNotExceededWithMethod() {
      String actualGreeting = assertTimeout(ofMinutes(2), AssertionShowTest::greeting);
      assertEquals("Hello, World!", actualGreeting);
  }

  private static String greeting() {
    return "Hello, World!";
  }

}
```

Now let's see JUnit Jupiter principal test case annotations and life cycle:

```java
package com.jos.dem.junit;

import static org.junit.jupiter.api.Assertions.assertTrue;

import java.util.logging.Logger;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInfo;

class StandardTest {

  private static final Logger log = Logger.getLogger(StandardTest.class.getName());

  @BeforeAll
  static void setup() {
    log.info("Before any test execution");
  }

  @BeforeEach
  void init() {
    log.info("Before each test execution");
  }

  @Test
  @DisplayName("succeed test case")
  void succeedingTest(TestInfo testInfo) {
    log.info(String.format("Running %s ...", testInfo.getDisplayName()));
    assertTrue(true, () -> "Always passing this test");
  }

  @AfterEach
  void finish() {
    log.info("After each test execution");
  }

  @AfterAll
  static void tearDown() {
    log.info("After any test execution");
  }

}
```

Please, notice `TestInfo` and how is it used to inject information about the current test as method argument. Also notice how in Junit 5 neither test classes nor test methods need to be public.

**Assumptions**

JUnit Jupiter or Junit 5 comes with a subset of the assumption methods and are used to run tests only if certain conditions are met, this is typically used for external conditions that are required for the test to run properly.


```java
package com.jos.dem.junit;

import static org.junit.jupiter.api.Assumptions.assumeTrue;
import static org.junit.jupiter.api.Assumptions.assumingThat;
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;

import java.util.logging.Logger;

class AssumptionShowTest {

  private final Logger log = Logger.getLogger(AssumptionShowTest.class.getName());
  private final Person person = new Person("josdem", "joseluis.delacruz@gmail.com");

  @Test
  @DisplayName("Assume that is a Gmail account")
  void shouldKnowIfIsGmailAccount(){
    assumeTrue(person.getEmail().endsWith("gmail.com"));
    log.info("Test continues ...");
    assertEquals("josdem", person.getNickname());
  }

   @Test
   @DisplayName("Assuming something based in conditions")
   void testAssumingThat() {
     assumingThat(2 > 1, () -> log.info("This should happen!"));
     assumingThat(2 < 1, () -> log.info("This should never happen!"));
   }

}
```

To download the code:

```bash
git clone https://github.com/josdem/junit5-workshop.git
```

To run the project using Gradle:

```bash
gradle test
```

To run the project using Maven:

```bash
mvn test
```


[Return to the main article](/techtalk/java)

