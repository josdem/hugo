+++
title =  "Selenium Cucumber And TestNG"
description = "Selenium Cucumber TestNG for testing web sites"
date = "2023-02-26T18:04:28-05:00"
tags = ["josdem", "techtalks","programming","technology", "automation", "selenium"]
categories = ["techtalk", "code", "ui", "automation"]
+++

[Selenium](https://www.selenium.dev/) is a collection of tools and libraries that enable and support automation for web browsers, Selenium follows the [W3c WebDriver Specification](https://www.w3.org/TR/webdriver/) standard, that is importat due to allows you to create interchangeable code for all major web browsers. Also is a truly open source project under Apache version 2.0 license. This time we will review how to build a broswer validation using Selenuium, [Cucumber](https://cucumber.io/) and [TestNG](https://testng.org/doc/). First, let's create a new Maven project using [IntelliJ](https://www.jetbrains.com/idea/) and select Java 17 version, your pom.xml file should looks like this:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.josdem.selenium.cucumber</groupId>
    <artifactId>selenium-cucumber</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <java.version>17</java.version>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>


</project>
```

Now, let's add selenium-java, cucumber-java, cucumber-testng and cucumber-core.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.josdem.selenium.cucumber</groupId>
    <artifactId>selenium-cucumber</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <selenium-java-version>4.8.0</selenium-java-version>
        <testng-version>7.7.1</testng-version>
        <cucumber-version>7.11.1</cucumber-version>
        <java.version>17</java.version>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>${selenium-java-version}</version>
        </dependency>
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <version>${testng-version}</version>
        </dependency>
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-java</artifactId>
            <version>${cucumber-version}</version>
        </dependency>
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-testng</artifactId>
            <version>${cucumber-version}</version>
        </dependency>
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-core</artifactId>
            <version>${cucumber-version}</version>
        </dependency>
    </dependencies>


</project>
```

In the same level as pom.xml, let's create a new file named `testng.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE suite SYSTEM "https://testng.org/testng-1.0.dtd">
<suite name="Test Suite" verbose="1">
    <test name="Regression Tests">
        <classes>
            <class name="com.josdem.selenium.cucumber.TestRunner">
            </class>
        </classes>
    </test>
</suite>
```

As you can see our group id is: `com.josdem.selenium.cucumber` and we need a Java class in src/test/java named `TestRunner` with this structure:

```java
package com.josdem.selenium.cucumber;

import io.cucumber.testng.AbstractTestNGCucumberTests;
import io.cucumber.testng.CucumberOptions;

@CucumberOptions(
    plugin = {
      "pretty",
      "html:target/cucumber-reports/cucumber.html",
      "json:target/cucumber-reports/cucumber.json"
    },
    features = {"src/test/resources/features"},
    glue = {"com.josdem.selenium.cucumber.steps"},
    tags = "@smoke")
public class TestRunner extends AbstractTestNGCucumberTests {}
```

That's it in this file we are defining our feature home, let's create a simple feature file.

```gherkin
@smoke
Feature: josdem Test
  Scenario: Validates josdem website title
    Given Website as "https://josdem.io/"
    When I get title
    Then I validate is "josdem"
```

And steps goes in com.josdem.selenium.cucumber.steps

```java
package com.josdem.selenium.cucumber.steps;

import io.cucumber.java.en.Given;
import io.cucumber.java.en.Then;
import io.cucumber.java.en.When;
import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;

import static org.testng.Assert.assertTrue;

public class WebsiteStep {

  private WebDriver driver = new ChromeDriver();
  private String title;

  @Given("^Website as \"(.*)\"$")
  public void loadingWebsite(String website) {
    driver.get(website);
  }

  @When("^I get title$")
  public void getTitle() {
    driver.manage().window().maximize();
    title = driver.getTitle();
  }

  @Then("^I validate is \"(.*)\"$")
  public void validateTitle(String name) {
    assertTrue(title.equals(name));
    driver.close();
  }
}
```

As you can see we are using regular expressions to pass arguments or values from feature file to the step definitions. Finally you can execute this project with command:

```bash
mvn test
```

To browse the code go [here](https://github.com/josdem/selenium-cucumber), to download the project:

```bash
git clone git@github.com:josdem/selenium-cucumber.git
```

[Return to the main article](/techtalk/ux)
