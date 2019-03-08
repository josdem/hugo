+++
title =  "Appium Cucumber and Junit5"
description = "Appium is an open-source tool for automating native, mobile web, and hybrid applications on both iOS and Android. In this post we will review how to do feature testing using Appium Cucumber and Junit5"
date = "2018-10-27T20:20:04-04:00"
tags = ["josdem", "techtalks","programming","technology","appium","appium cucumber","appium junit5"]
categories = ["techtalk", "code","android"]
+++

[Appium](http://appium.io/) is an open-source tool for automating native, mobile web, and hybrid applications on both iOS and Android. In this post we will review how to do feature testing using Appium, [Cucumber](https://cucumber.io/) and [Junit5](https://junit.org/junit5/). **NOTE:** If you want to know how to setup Appium, please refer my previous post: [Appium Automation](/techtalk/android/appium_automation)

First, create a new project in android studio with no Activity, default options and set the following dependencies in `build.gradle`

```groovy
apply plugin: 'com.android.application'

ext {
  cucumberVersion = '1.2.5'
  appiumJavaClient = '6.1.0'
  junitJupiterVersion = '5.3.1'
}

android {
  compileSdkVersion 28
  defaultConfig {
    applicationId "com.jos.dem.appium.service"
    minSdkVersion 26
    targetSdkVersion 26
    versionCode 1
    versionName "1.0"
    testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
  }
  buildTypes {
    release {
      minifyEnabled false
      proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
  }
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}

dependencies {
  implementation 'com.android.support:appcompat-v7:28.0.0-rc01'
  implementation 'com.android.support.constraint:constraint-layout:1.1.2'
  implementation 'com.android.support:design:28.0.0-rc01'
  testImplementation "org.junit.jupiter:junit-jupiter-api:$junitJupiterVersion"
  testImplementation "org.junit.jupiter:junit-jupiter-engine:$junitJupiterVersion"
  testImplementation "io.appium:java-client:$appiumJavaClient"
  androidTestImplementation 'com.android.support.test:runner:1.0.2'
  implementation 'com.android.support.test.espresso:espresso-core:3.0.2'
  implementation 'org.apache.commons:commons-lang3:3.8.1'
  implementation "info.cukes:cucumber-java:$cucumberVersion"
  implementation "info.cukes:cucumber-junit:$cucumberVersion"
  implementation 'org.apache.commons:commons-configuration2:2.4'
  implementation 'commons-beanutils:commons-beanutils:1.9.3'
}
```

Now, lets create a service to take care about Android capabilities.

```java
package com.jos.dem.appium.service;

import java.io.IOException;
import org.openqa.selenium.remote.DesiredCapabilities;

public interface AppiumService {

  void setCapabilities(DesiredCapabilities capabilities) throws IOException;

}
```

Here is our service implementation:

```java
package com.jos.dem.appium.service.impl;

import java.io.File;
import java.io.IOException;

import org.openqa.selenium.remote.DesiredCapabilities;

import com.jos.dem.appium.util.ConfigurationReader;
import com.jos.dem.appium.service.AppiumService;

public class AppiumServiceImpl implements AppiumService {

  public void setCapabilities(DesiredCapabilities capabilities) throws IOException {
    capabilities.setCapability("deviceName", ConfigurationReader.getProperty("device.name"));
    capabilities.setCapability("platformName", ConfigurationReader.getProperty("device.platform"));
    capabilities.setCapability("platformVersion", ConfigurationReader.getProperty("device.version"));
    capabilities.setCapability("appPackage", ConfigurationReader.getProperty("application.package"));
    capabilities.setCapability("appActivity", ConfigurationReader.getProperty("application.activity"));
  }

}
```

Desired Capabilities are keys and values encoded in a JSON object, sent by Appium clients to the server when a new automation session is requested. Here is our `$PROJECT_HOME/app/src/main/res/configuration.properties` file.

```properties
device.name=Google Pixel
device.version=9
device.platform=Android
application.package=com.jugoterapia.josdem
application.activity=com.jugoterapia.josdem.activity.CategoryActivity
appium.server=http://127.0.0.1:4723/wd/hub
appium.wait=10
appium.sleep=2
appium.timeout=20
```

**Note:** If you want to know more about Appium capabilities go [here](http://appium.io/docs/en/writing-running-appium/caps/). If you want to know how to read a properties file using Apache Commons, please go to my technical post [here](/techtalk/java/configuration_apache_commons/). The JUnit runner uses the JUnit framework to run Cucumber test cases, so we need is to create a single empty class with an annotation `@RunWith(Cucumber.class)` and define `@CucumberOptions` where we’re specifying the location of the Gherkin file which is also known as the feature file:

```java
package com.jos.dem.appium;

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
package com.jos.dem.appium.step;

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

import io.appium.java_client.android.AndroidDriver;
import io.appium.java_client.android.AndroidElement;
import io.appium.java_client.android.nativekey.KeyEvent;
import io.appium.java_client.android.nativekey.AndroidKey;

import com.jos.dem.appium.util.ConfigurationReader;

public class JugoterapiaStep extends BaseStep {

  private AndroidElement textView;
  private AndroidDriver<AndroidElement> driver;
  private Long timeToSleep = Long.parseLong(ConfigurationReader.getProperty("appium.sleep"));

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
    textView = driver.findElement(By.id("beverageTextView"));

    log.info("Beverages container and beverage list are there");
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
package com.jos.dem.appium.step;

import java.net.URL;
import java.util.concurrent.TimeUnit;
import java.io.IOException;

import org.openqa.selenium.support.ui.WebDriverWait;
import org.openqa.selenium.support.ui.ExpectedConditions;
import org.openqa.selenium.remote.DesiredCapabilities;

import io.appium.java_client.android.AndroidDriver;
import io.appium.java_client.android.AndroidElement;

import com.jos.dem.appium.service.AppiumService;
import com.jos.dem.appium.service.impl.AppiumServiceImpl;
import com.jos.dem.appium.util.ConfigurationReader;

public class BaseStep {

  private static AndroidDriver<AndroidElement> driver;
  private static AppiumService appiumService = new AppiumServiceImpl();
  private static DesiredCapabilities capabilities = new DesiredCapabilities();

  public static AndroidDriver<AndroidElement> getDriver() throws IOException {
    if(driver == null){
      appiumService.setCapabilities(capabilities);
      driver = new AndroidDriver(new URL(ConfigurationReader.getProperty("appium.server")), capabilities);
      driver.manage().timeouts().implicitlyWait(Long.parseLong(ConfigurationReader.getProperty("appium.wait")), TimeUnit.SECONDS);
    }
    return driver;
  }

  public static AndroidElement waitForElement(AndroidElement element){
    WebDriverWait wait =  new WebDriverWait(driver, Long.parseLong(ConfigurationReader.getProperty("appium.timeout")));
    wait.until(ExpectedConditions.visibilityOf(element));
    return element;
  }

  public static void stopDriver(){
    driver.quit();
  }

}
```

**Note:** This project uses Jugoterapia project as an Android application for testing, download it from Google Play Store [here](https://play.google.com/store/apps/details?id=com.jugoterapia.josdem), to know more about this project go [here](http://josdem.io/jugoterapia/jugoterapia/). Now you can go to the `$PROJECT_HOME` and run:

```bash
gradle testDebug
```

Video

<iframe width="300" height="550" src="https://www.youtube.com/embed/rlxkgI0WE8s" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>


To browse the code go [here](https://github.com/josdem/jugoterapia-appium), to download the project:

```bash
git clone git@github.com:josdem/jugoterapia-appium.git
```

[Return to the main article](/techtalk/android)
