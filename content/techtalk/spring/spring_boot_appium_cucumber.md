+++
title =  "Spring Boot Appium & Cucumber"
description = "Appium is an open-source tool for automating native, mobile web, and hybrid applications on both iOS and Android. In this post we will review how to do feature testing using Spring Boot, Appium, Cucumber and Junit5"
date = "2018-11-15T15:28:48-05:00"
tags = ["josdem", "techtalks","programming","technology","appium","cucumber","junit5","spring boot"]
categories = ["techtalk", "code","spring boot","appium"]
+++

[Appium](http://appium.io/) is an open-source tool for automating native, mobile web, and hybrid applications on both iOS and Android. In this post we will review how to do feature testing using Spring Boot, Appium, [Cucumber](https://cucumber.io/) and [Junit5](https://junit.org/junit5/). **NOTE:** If you need to know what tools you need to have installed in your computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot)

Then execute this command in your terminal.

```bash
spring init --dependencies=webflux,lombok --language=java --build=gradle spring-boot-appium-jugoterapia
```

This is the `build.gradle file` generated:

```groovy
buildscript {
  ext {
    springBootVersion = '2.1.0.RELEASE'
  }
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  }
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.appium.jugoterapia'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = 1.8

repositories {
  mavenCentral()
}

dependencies {
  implementation('org.springframework.boot:spring-boot-starter-webflux')
  compileOnly('org.projectlombok:lombok')
  testImplementation('org.springframework.boot:spring-boot-starter-test')
  testImplementation('io.projectreactor:reactor-test')
}
```

Now add latest Appium, Cucumber and Junit 5 Framework dependencies to your build.gradle file:

```groovy
implementation "info.cukes:cucumber-java:$cucumberVersion"
implementation "info.cukes:cucumber-junit:$cucumberVersion"
implementation "info.cukes:cucumber-spring:$cucumberVersion"
implementation "io.appium:java-client:$appiumJavaClient"
testImplementation "org.junit.jupiter:junit-jupiter-api:$junitJupiterVersion"
testImplementation "org.junit.jupiter:junit-jupiter-engine:$junitJupiterVersion"
```

Now, lets create an service to take care about Android capabilities.

```java
package com.jos.dem.springboot.appium.jugoterapia.service;

import org.openqa.selenium.remote.DesiredCapabilities;

public interface AppiumService {
  DesiredCapabilities getCapabilities();
}
```

Here is the implementation:

```java
package com.jos.dem.springboot.appium.jugoterapia.service.impl;

import javax.annotation.PostConstruct;

import org.openqa.selenium.remote.CapabilityType;
import org.openqa.selenium.remote.DesiredCapabilities;

import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Value;

import com.jos.dem.springboot.appium.jugoterapia.service.AppiumService;

@Service
public class AppiumServiceImpl implements AppiumService {

  @Value("${device.name}")
  private String deviceName;
  @Value("${device.version}")
  private String deviceVersion;
  @Value("${device.platform}")
  private String devicePlatform;
  @Value("${application.package}")
  private String applicationPackage;
  @Value("${application.activity}")
  private String applicationActivity;

  private DesiredCapabilities capabilities = new DesiredCapabilities();

  @PostConstruct
  public void setup(){
    capabilities.setCapability("deviceName", deviceName);
    capabilities.setCapability(CapabilityType.VERSION, deviceVersion);
    capabilities.setCapability("platformName", devicePlatform);
    capabilities.setCapability("appPackage", applicationPackage);
    capabilities.setCapability("appActivity", applicationActivity);
  }

  public DesiredCapabilities getCapabilities(){
    return capabilities;
  }

}
```

Here is our `application.properties`

```properties
device.name=Pixel 2
device.version=9
device.platform=Android
application.package=com.jugoterapia.josdem
application.activity=com.jugoterapia.josdem.activity.CategoryActivity
appium.server=http://127.0.0.1:4723/wd/hub
appium.wait=10
appium.timeout=20
appium.sleep=2
```

Desired Capabilities are keys and values encoded in a JSON object, sent by Appium clients to the server when a new automation session is requested. The JUnit runner uses the JUnit framework to run the Cucumber Test. What we need is to create a single empty class with an annotation `@RunWith(Cucumber.class)` and define `@CucumberOptions` where we’re specifying the location of the Gherkin file which is also known as the feature file:

```java
package com.jos.dem.springboot.appium.jugoterapia;

import org.junit.runner.RunWith;

import cucumber.api.junit.Cucumber;
import cucumber.api.CucumberOptions;

@RunWith(Cucumber.class)
@CucumberOptions(features = "src/test/resources")
public class CucumberTest {}
```

Gherkin is a DSL language used to describe an application feature that needs to be tested. Here is our category Gherkin feature definition file: `src/test/resources/jugoterapia.feature`

```gherkin
Feature: Jugoterapia run an end-to-end user flow
  Scenario: As a user I should be able to select a category, beverage and recipe
    When I launch the application
    Then I should be able to see the category list
      And I should be able to click in the category
      And I should be able to list beverages
      And I should be able to click in a beverage
      And I should be able to view a recipe
      And I should back to beverage section
      And I should back to category section
      And I should be able to close application
```

Now let’s create a test case in the Java to represent to this feature scenarios

```java
package com.jos.dem.springboot.appium.jugoterapia;

import static org.junit.jupiter.api.Assumptions.assumeTrue;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertNotNull;

import org.openqa.selenium.By;
import org.openqa.selenium.support.ui.Sleeper;
import org.openqa.selenium.remote.CapabilityType;
import org.openqa.selenium.remote.DesiredCapabilities;

import java.net.URL;
import java.time.Duration;
import java.util.List;
import java.util.Date;
import java.util.concurrent.TimeUnit;
import java.util.logging.Logger;

import cucumber.api.java.en.And;
import cucumber.api.java.en.When;
import cucumber.api.java.en.Then;

import org.springframework.beans.factory.annotation.Value;

import io.appium.java_client.android.AndroidDriver;
import io.appium.java_client.android.AndroidElement;
import io.appium.java_client.android.nativekey.KeyEvent;
import io.appium.java_client.android.nativekey.AndroidKey;

public class JugoterapiaTest extends BaseTest {

  @Value("${appium.sleep}")
  private Long timeToSleep;

  private AndroidElement textView;
  private AndroidDriver<AndroidElement> driver;

  private Logger log = Logger.getLogger(this.getClass().getName());

  @When("I launch the application")
  public void shouldLaunchTheApplication() throws Exception {
    log.info("Running: I launch the application at " + new Date());
    driver = getDriver();
  }

  @Then("I should be able to see the category list")
  public void shouldDisplayCategories() throws Exception {
    log.info("Running: I should be able to see the category list at " + new Date());
    assumeTrue(driver.findElement(By.id("listViewCategories")) != null);
    textView = driver.findElement(By.id("categoryTextView"));
    assertEquals("Curativos", textView.getText());
  }

  @And("I should be able to click in the category")
  public void shouldClickInCategory() throws Exception {
    log.info("Running: I should be able to click in the category at " + new Date());
    waitForElement(textView).click();
  }

  @And("I should be able to list beverages")
  public void shouldListBeverages() throws Exception {
    log.info("Running: I should be able to list beverages at " + new Date());
    assertNotNull(driver.findElement(By.id("action_bar_container")));
    assumeTrue(driver.findElement(By.id("content")) != null);
    assumeTrue(driver.findElement(By.id("listViewBeverages")) != null);

    log.info("Beverages container and beverage list are there");
    textView = driver.findElement(By.id("beverageTextView"));
    assertEquals("Jugo para evitar los calambres", textView.getText());
  }

  @And("I should be able to click in a beverage")
  public void shouldClickInBeverage() throws Exception {
    log.info("Running: I should be able to click in a beverage at " + new Date());
    waitForElement(textView).click();
  }

  @And("I should be able to view a recipe")
  public void shouldViewRecipe() throws Exception {
    log.info("Running: I should be able to view a recipe at " + new Date());
    assertNotNull(driver.findElement(By.id("name")));
    assertNotNull(driver.findElement(By.id("image")));
    assertNotNull(driver.findElement(By.id("ingredients")));
    assertNotNull(driver.findElement(By.id("recipe")));
  }

  @And("I should back to beverage section")
  public void shouldBackToBeverageSection() throws Exception {
    log.info("Running: I should back to the beverage section at " + new Date());
    driver.pressKey(new KeyEvent(AndroidKey.BACK));
    Sleeper.SYSTEM_SLEEPER.sleep(Duration.ofSeconds(timeToSleep));
  }

  @And("I should back to category section")
  public void shouldBackToCategorySection() throws Exception {
    log.info("Running: I should back to the category section at " + new Date());
    driver.pressKey(new KeyEvent(AndroidKey.BACK));
    Sleeper.SYSTEM_SLEEPER.sleep(Duration.ofSeconds(timeToSleep));
  }

  @And("I should be able to close application")
  public void shouldCloseTheApplication() throws Exception {
    log.info("Running: I should be able to close the application at " + new Date());
    driver.pressKey(new KeyEvent(AndroidKey.BACK));
    Sleeper.SYSTEM_SLEEPER.sleep(Duration.ofSeconds(timeToSleep));
  }

}
```

`BaseTest` has Appium and AppiumDriver management, in that way we can delegate in our test cases Appium commands.

```java
package com.jos.dem.springboot.appium.jugoterapia;

import java.net.URL;
import java.util.concurrent.TimeUnit;
import java.net.MalformedURLException;

import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.remote.DesiredCapabilities;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.web.WebAppConfiguration;

import io.appium.java_client.android.AndroidDriver;
import io.appium.java_client.android.AndroidElement;

import com.jos.dem.springboot.appium.jugoterapia.service.AppiumService;

@ContextConfiguration(classes = AppiumJugoterapiaApplication.class)
@WebAppConfiguration
public class BaseTest {

  @Value("${appium.server}")
  private String appiumServer;
  @Value("${appium.wait}")
  private Long appiumWait;
  @Value("${appium.timeout}")
  private Long appiumTimeout;

  @Autowired
  private AppiumService appiumService;
  private AndroidDriver<AndroidElement> driver;

  public AndroidDriver<AndroidElement> getDriver() throws MalformedURLException {
    if(driver == null){
      driver = new AndroidDriver(new URL(appiumServer), appiumService.getCapabilities());
      driver.manage().timeouts().implicitlyWait(appiumWait, TimeUnit.SECONDS);
    }
    return driver;
  }

  public AndroidElement waitForElement(AndroidElement element){
    WebDriverWait wait =  new WebDriverWait(driver, appiumTimeout);
    wait.until(ExpectedConditions.visibilityOf(element));
    return element;
  }

  DesiredCapabilities getCapabilities(){
    return appiumService.getCapabilities();
  }

}
```

**Note:** This project uses Jugoterapia project as an Android application for testing, download it from Google Play Store [here](https://play.google.com/store/apps/details?id=com.jugoterapia.josdem), to know more about this project go [here](http://josdem.io/jugoterapia/jugoterapia/). Now you can go to the `$PROJECT_HOME` and run:

```bash
gradle test
```

If you prefer Maven

```bash
mvn test
```

Video

<iframe width="300" height="550" src="https://www.youtube.com/embed/rlxkgI0WE8s" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>


To browse the code go [here](https://github.com/josdem/spring-boot-appium-jugoterapia), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-appium-jugoterapia.git
```

[Return to the main article](/techtalk/spring#Spring_Boot)
