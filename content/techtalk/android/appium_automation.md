+++
title =  "Appium Automation"
description = "Appium is an open source test automation framework for use in mobile apps. It drives iOS, Android using the WebDriver protocol"
date = "2018-10-19T11:13:51-04:00"
tags = ["josdem", "techtalks","programming","technology","android","appium","automation"]
categories = ["techtalk", "code"]
+++

Appium is an open-source tool for automating native, mobile web, and hybrid applications on both iOS and Android. Appium supports app automation across a variety of platforms, like iOS, Android, and Windows. Each platform is supported by one or more "drivers", which know how to automate that particular platform.

**Appium Philosophy**

Appium was designed to meet mobile automation needs according to a philosophy outlined by the following four tenets:

* You shouldn't have to recompile your app or modify it in any way in order to automate it.
* You shouldn't be locked into a specific language or framework to write and run your tests.
* You can use any testing framework.
* A mobile automation framework shouldn't reinvent the wheel when it comes to automation APIs.
  * WebDriver model that's already the standard for web automation.
  * Appium is using REST API protocol

**Appium Architecture**

<img src="/img/techtalks/android/appium_architecture.png">

**Appium Server**

Appium is a server written in Node.js. It can be built and installed from source or installed directly from npm.

```bash
~ npm install -g appium
~ appium
```

**Appium Clients**

There are client libraries (in Java, Ruby, Python, PHP, JavaScript, and C#) which support Appium's extensions to the WebDriver protocol. When using Appium, you want to use these client libraries instead of your regular [WebDriver](https://w3c.github.io/webdriver/) client. You can view the full list of libraries [here](http://appium.io/docs/en/about-appium/appium-clients/index.html).

**Verifying the Installation**

To verify that all of Appium's dependencies are met you can use appium-doctor. Install it with:

```bash
~ npm install -g appium-doctor
~ appium-doctor
```

*Output*

```bash
info AppiumDoctor Appium Doctor v.1.4.3
info AppiumDoctor ### Diagnostic starting ###
info AppiumDoctor  ✔ The Node.js binary was found at: /usr/local/bin/node
info AppiumDoctor  ✔ Node version is 10.8.0
info AppiumDoctor  ✔ Xcode is installed at: /Library/Developer/CommandLineTools
info AppiumDoctor  ✔ Xcode Command Line Tools are installed.
info AppiumDoctor  ✔ DevToolsSecurity is enabled.
info AppiumDoctor  ✔ The Authorization DB is set up properly.
info AppiumDoctor  ✔ Carthage was found at: /usr/local/bin/carthage
info AppiumDoctor  ✔ HOME is set to: /Users/josdem
info AppiumDoctor  ✔ ANDROID_HOME is set to: /Users/josdem/Library/Android/sdk
info AppiumDoctor  ✔ JAVA_HOME is set to: /Users/josdem/Java/jdk-11.jdk/Contents/Home
info AppiumDoctor  ✔ adb exists at: /Users/josdem/Library/Android/sdk/platform-tools/adb
info AppiumDoctor  ✔ android exists at: /Users/josdem/Library/Android/sdk/tools/android
info AppiumDoctor  ✔ emulator exists at: /Users/josdem/Library/Android/sdk/tools/emulator
info AppiumDoctor  ✔ Bin directory of $JAVA_HOME is set
info AppiumDoctor ### Diagnostic completed, no fix needed. ###
info AppiumDoctor
info AppiumDoctor Everything looks good, bye!
info AppiumDoctor
```

You can browse the code [here](https://github.com/josdem/jugoterapia-appium), you can download the code here:

```bash
git clone git@github.com:josdem/jugoterapia-appium.git
```

[Return to the main article](/techtalk/android)
