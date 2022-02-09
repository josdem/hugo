+++
title =  "WebdriverIO with Cucumber Getting Started"
description = "Getting started with Webdriverio with Cucumber basic configuration"
date = "2022-02-09T13:27:24-05:00"
tags = ["josdem", "techtalks","programming","technology","WebdriverIO"]
categories = ["techtalk", "code", "NodeJS", "JavaScript","WebdriverIO"]
+++

[WebdriverIO](https://webdriver.io/) Is an automation framework based in [NodeJS](https://nodejs.org/en/) designed to support cross-browser testing and modern mobile native applications. It's archictecture is based on plugins you can use to extend easily functionlality. WebdriverIO rely on [WebDriver](https://w3c.github.io/webdriver/) protocols that guarantees a true cross-browsing experience and also is a truly open source project driven by [OpenJSFoundation](https://openjsf.org/). This time we will connect WebdriverIO with [Cucumber Framework](https://cucumber.io/). Let's start by installing NodeJS in our computer, I highly recommend to use [NVM](https://github.com/nvm-sh/nvm) so you can manage different versions. Once installed let's create a new directory named `webdriverio-cucumber-getting-started` and execute this command inside that new directory:

```bash
npm init
```

Answer general questions about this project coming in the prompt line, so you can have a `package.json` generated very similar to this one:

```javascript
{
  "name": "webdriverio-cucumber-getting-started",
  "version": "1.0.0",
  "description": "A cool automation framework with Cucumber",
  "main": "index.js",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/josdem/webdriverio-cucumber-getting-started.git"
  },
  "keywords": [
    "webdriverio"
  ],
  "author": "josdem",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/josdem/webdriverio-cucumber-getting-started/issues"
  },
  "homepage": "https://github.com/josdem/webdriverio-cucumber-getting-started#readme"
}
```

Next step is to install WebdriverIO [CLI](https://www.npmjs.com/package/@wdio/cli) which will allow us to use command line WendriverIO interface.


```bash
npm i --save-dev @wdio/cli
```

Now it is time to configure WebdriverIO adding some dependencies:

```bash
@wdio/local-runner
@wdio/cucumber-framework
@wdio/spec-reporter
chromedriver
wdio-chromedriver-service
```

in order to do that, type this command:

```bash
npx wdio config
```

And answer prompt questions with:

- Where is your automation backend located? -> On my local machine
- Which framework do you want to use? -> cucumber
- Do you want to use a compiler? -> No
- Where are your test specs located? -> `./features/step-definitions/steps.js`
- Do you want WebdriverIO to autogenerate some test files? - No
- Which reporter do you want to use? -> spec
- Do you want to add a plugin to your test setup? wait-for
- Do you want to add a service to your test setup? -> chromedriver
- What is the base url? -> http://localhost

This will gererate a WebdriverIO configuration file named `wdio.conf.js`. Finally we can create our first test case, let's create a feature file named `helloWorld.feature` under `${PROJECT_HOME}/features/helloWorld.feature`

```gherkin
Feature: To test a website

Scenario: Loading webpage
  Given A webpage as "https://josdem.io/"
  When I get page title
  Then I validate title is "josdem"
```

Inside the `features` folder create a `step-definitions/steps.js` file with this content:

```javascript
const { Given, When, Then } = require('@cucumber/cucumber')
```

After that we should be good to create our steps definition with this command:

```bash
./node_modules/.bin/cucumber-js
```

Copy generated code (step definition methods) and paste them into the `steps.js` file. Your file should look like this:

```javascript
const { Given, When, Then } = require('@cucumber/cucumber')

Given('A webpage as {string}', function (string) {
  // Write code here that turns the phrase above into concrete actions
  return 'pending;
})


When('I get page title', function () {
  // Write code here that turns the phrase above into concrete actions
  return 'pending'
})

Then('I validate title is {string}', function (string) {
  // Write code here that turns the phrase above into concrete actions
  return 'pending'
});
```

Finally add your steps implementation


```javascript
const { Given, When, Then } = require('@cucumber/cucumber')

let title

Given('A webpage as {string}', function (keyword) {
    browser.url(keyword)
})

When('I get page title', function () {
    title = browser.getTitle()
})

Then('I validate title is {string}', function (title) {
    expect(title === "josdem").toBeTruthy()
})
```

You are good to execute it with: `npx wdio run wdio.conf.js`, and you should see this output:

```bash
 "spec" Reporter:
------------------------------------------------------------------
[chrome 98.0.4758.80 linux #0-0] Running: chrome (v98.0.4758.80) on linux
[chrome 98.0.4758.80 linux #0-0] Session ID: 3176c7b69b896934eaf6d4dbb65939ea
[chrome 98.0.4758.80 linux #0-0]
[chrome 98.0.4758.80 linux #0-0] » /features/helloWorld.feature
[chrome 98.0.4758.80 linux #0-0] To test a website
[chrome 98.0.4758.80 linux #0-0] Loading webpage
[chrome 98.0.4758.80 linux #0-0]    ✓ Given A webpage as "https://josdem.io/"
[chrome 98.0.4758.80 linux #0-0]    ✓ When I get page title
[chrome 98.0.4758.80 linux #0-0]    ✓ Then I validate title is "josdem"
[chrome 98.0.4758.80 linux #0-0]
[chrome 98.0.4758.80 linux #0-0] 3 passing (1s)


Spec Files:	 1 passed, 1 total (100% completed) in 00:00:02
```


To browse the code go [here](https://github.com/josdem/webdriverio-getting-started), to download the project:

```bash
git clone git@github.com:josdem/webdriverio-getting-started.git
```

[Return to the main article](/techtalk/ux)
