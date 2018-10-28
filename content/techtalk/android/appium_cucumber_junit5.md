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
  implementation fileTree(include: ['*.jar'], dir: 'libs')
  implementation 'com.android.support:appcompat-v7:28.0.0-rc01'
  implementation 'com.android.support.constraint:constraint-layout:1.1.2'
  implementation 'com.android.support:design:28.0.0-rc01'
  testImplementation "org.junit.jupiter:junit-jupiter-api:$junitJupiterVersion"
  testImplementation "org.junit.jupiter:junit-jupiter-engine:$junitJupiterVersion"
  androidTestImplementation 'com.android.support.test:runner:1.0.2'
  implementation 'com.android.support.test.espresso:espresso-core:3.0.2'
  implementation 'org.apache.commons:commons-lang3:3.8.1'
  implementation "info.cukes:cucumber-java:$cucumberVersion"
  implementation "info.cukes:cucumber-junit:$cucumberVersion"
  implementation "io.appium:java-client:$appiumJavaClient"
  implementation files('libs/byte-buddy-1.8.15.jar')
  implementation files('libs/commons-codec-1.10.jar')
  implementation files('libs/commons-exec-1.3.jar')
  implementation files('libs/commons-logging-1.2.jar')
  implementation files('libs/httpclient-4.5.5.jar')
  implementation files('libs/httpcore-4.4.9.jar')
  implementation files('libs/okhttp-3.10.0.jar')
  implementation files('libs/okio-1.14.1.jar')
  implementation files('libs/client-combined-3.14.0.jar')
}
```

**NOTE:** In order to get Java Selenium Client & WebDriver, please download it from [here](https://www.seleniumhq.org/download/) then copy the required jar files in `$PROJECT_HOME/app/libs`

Now, lets create an service to take care about Android capabilities.

```java
package com.jos.dem.appium.service;

import org.openqa.selenium.remote.DesiredCapabilities;

public interface CategoryService {
  void setCapabilities(DesiredCapabilities capabilities);
}
```

Here is the implementation

```java
package com.jos.dem.appium.service.impl;

import org.openqa.selenium.remote.CapabilityType;
import org.openqa.selenium.remote.DesiredCapabilities;

import com.jos.dem.appium.service.CategoryService;

public class CategoryServiceImpl implements CategoryService {

  public void setCapabilities(DesiredCapabilities capabilities){
    capabilities.setCapability("deviceName", "Pixel 2");
    capabilities.setCapability(CapabilityType.VERSION, "9");
    capabilities.setCapability("platformName", "Android");
    capabilities.setCapability("appPackage", "com.jugoterapia.josdem");
    capabilities.setCapability("appActivity", "com.jugoterapia.josdem.activity.CategoryActivity");
  }

}
```

Desired Capabilities are keys and values encoded in a JSON object, sent by Appium clients to the server when a new automation session is requested. The JUnit runner uses the JUnit framework to run the Cucumber Test. What we need is to create a single empty class with an annotation @RunWith(Cucumber.class) and define @CucumberOptions where we’re specifying the location of the Gherkin file which is also known as the feature file:

```java
package com.jos.dem.appium;

import org.junit.runner.RunWith;

import cucumber.api.junit.Cucumber;
import cucumber.api.CucumberOptions;

@RunWith(Cucumber.class)
@CucumberOptions(features = "src/test/resources")
public class CucumberTest {}
```

Gherkin is a DSL language used to describe an application feature that needs to be tested. Here is our category Gherkin feature definition file: `src/test/resources/Cateogry.feature`

```gherkin
Feature: Display all categories
  Scenario: As a user I should be able to display all categories
    When I launch the application
    Then I should be able to see the category list
      And I should be able to click in the category
```

Now let’s create the step in the Java to represent to this test case scenario

```java
package com.jos.dem.appium.step;

import static org.junit.jupiter.api.Assumptions.assumeTrue;
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.openqa.selenium.By;
import org.openqa.selenium.WebElement;
import org.openqa.selenium.remote.CapabilityType;
import org.openqa.selenium.remote.DesiredCapabilities;

import java.net.URL;
import java.util.List;
import java.util.Date;
import java.util.concurrent.TimeUnit;
import java.util.logging.Logger;

import cucumber.api.java.en.And;
import cucumber.api.java.en.When;
import cucumber.api.java.en.Then;
import cucumber.api.java.After;
import cucumber.api.java.Before;

import io.appium.java_client.AppiumDriver;
import io.appium.java_client.android.AndroidElement;

import com.jos.dem.appium.service.CategoryService;
import com.jos.dem.appium.service.impl.CategoryServiceImpl;

public class CategoryStep {

  private WebElement textView;
  private AppiumDriver<WebElement> driver;
  private DesiredCapabilities capabilities = new DesiredCapabilities();
  private CategoryService categoryService = new CategoryServiceImpl();

  private Logger log = Logger.getLogger(this.getClass().getName());

  @Before
  public void setup(){
    categoryService.setCapabilities(capabilities);
  }

  @When("I launch the application")
  public void shouldLaunchTheApplication() throws Exception {
    log.info("Running: I launch the application at " + new Date());
    driver = new AppiumDriver(new URL("http://127.0.0.1:4723/wd/hub"), capabilities);
    driver.manage().timeouts().implicitlyWait(30, TimeUnit.SECONDS);
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
    textView.click();
  }

  @After
  public void end(){
    driver.quit();
  }

}
```

**Note:** This project uses Jugoterapia project as an Android application for testing, download it from Google Play Store [here](https://play.google.com/store/apps/details?id=com.jugoterapia.josdem), to know more about this project go [here](http://josdem.io/jugoterapia/jugoterapia/). Now you can go to the `$PROJECT_HOME` and run:

```bash
gradle testDebug
```

This is the output:

```bash
Task :app:testDebugUnitTest
BUILD SUCCESSFUL in 28s
19 actionable tasks: 2 executed, 17 up-to-date
```

Video

<iframe width="300" height="550" src="https://www.youtube.com/embed/9uizmsQdg_4" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>


To browse the code go [here](https://github.com/josdem/jugoterapia-appium), to download the project:

```bash
git clone git@github.com:josdem/jugoterapia-appium.git
```

[Return to the main article](/techtalk/android)
