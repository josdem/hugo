+++
title =  "WebdriverIO Getting Started"
description = "Getting started with Webdriverio and configuration"
date = "2021-10-25T14:14:42-04:00"
tags = ["josdem", "techtalks","programming","technology","WebdriverIO"]
categories = ["techtalk", "code", "NodeJS", "JavaScript","WebdriverIO"]
+++

[WebdriverIO](https://webdriver.io/) Is an automation framework based in [NodeJS](https://nodejs.org/en/) designed to support cross-browser testing and modern mobile native applications. It's archictecture is based on plugins you can use to extend easily functionlality. WebdriverIO rely on [WebDriver](https://w3c.github.io/webdriver/) protocols that guarantees a true cross-browsing experience and also is a truly open source project driven by [OpenJSFoundation](https://openjsf.org/). Let's start by installing NodeJS in our computer, I highly recommend to use [NVM](https://github.com/nvm-sh/nvm) so you can manage different versions. Once installed let's create a new directory named `webdriverio-getting-started` and execute this command:

```bash
npm init
```

Answer general questions about this project coming in the prompt line, so you can have a `package.json` generated very similar to this one:

```javascript
{
  "name": "webdriverio-getting-started",
  "version": "1.0.0",
  "description": "A cool automation framework",
  "main": "index.js",
  "scripts": {
    "test": "npm test"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/josdem/webdriverio-getting-started.git"
  },
  "keywords": [
    "webdriverio"
  ],
  "author": "josdem",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/josdem/webdriverio-getting-started/issues"
  },
  "homepage": "https://github.com/josdem/webdriverio-getting-started#readme"
}
```

[Return to the main article](/techtalk/ux)
