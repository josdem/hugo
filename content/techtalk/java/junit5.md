+++
title =  "Junit 5"
description = "Junit5"
date = "2018-05-02T15:27:13-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code", "java"]
+++

Junit 5 is the next generation of Junit, it requires Java 8 since it was born as JUnit Lambda project, that project wanted to take full advantage of the new features from Java 8, especially lambda expressions.

Using Gradle, you need to create a `build.gradle` file with the following structure:

```groovy
apply plugin: "java"

repositories {
  mavenCentral()
}

dependencies {
  testCompile 'org.junit.jupiter:junit-jupiter-api:5.2.0'
  testRuntime 'org.junit.jupiter:junit-jupiter-engine:5.2.0'
}

test {
  useJUnitPlatform()

  reports {
    html.enabled = true
  }
}
```

Dependencies includes Junit Jupiter also know as Junit 5, configuring a `testCompile` dependency on the JUnit Jupiter API and a `testRuntime` dependency on the JUnit Jupiter TestEngine it is minimum required configuration. Also `test` task definition is supported since Gradle 4.6 version and specify Junit platform support, besides with `html.engine = true` we can generate html reports in a similar way `Spock Framework` does.

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

To test the code:

```bash
gradle test
```

[Return to the main article](/techtalk/java)

