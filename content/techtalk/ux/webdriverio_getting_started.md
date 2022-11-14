+++
title =  "WebdriverIO Getting Started"
description = "Getting started with Webdriverio and configuration"
date = "2021-10-25T14:14:42-04:00"
tags = ["josdem", "techtalks","programming","technology","WebdriverIO"]
categories = ["techtalk", "code", "NodeJS", "JavaScript","WebdriverIO"]
+++

[WebdriverIO](https://webdriver.io/) Is an automation framework based in [NodeJS](https://nodejs.org/en/) designed to support cross-browser testing and modern mobile native applications. It's archictecture is based on plugins you can use to extend easily functionlality. WebdriverIO rely on [WebDriver](https://w3c.github.io/webdriver/) protocols that guarantees a true cross-browsing experience and also is a truly open source project driven by [OpenJSFoundation](https://openjsf.org/). Let's start by installing NodeJS in our computer, I highly recommend to use [NVM](https://github.com/nvm-sh/nvm) so you can manage different versions. Once installed let's create a new directory named `webdriverio-getting-started` and execute this command inside that new directory:

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

Next step is to install WebdriverIO [CLI](https://www.npmjs.com/package/@wdio/cli) which will allow us to use command line WendriverIO interface.


```bash
npm i --save-dev @wdio/cli
```

Now it is time to configure WebdriverIO adding some dependencies:

```bash
@wdio/local-runner
@wdio/mocha-framework
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
- Which framework do you want to use? -> Mocha
- Do you want to use a compiler? -> No
- Where are your test specs located? -> `./test/specs/**/*.js`
- Do you want WebdriverIO to autogenerate some test files? - No
- Which reporter do you want to use? -> spec
- Do you want to add a service to your test setup? -> chromedriver
- What is the base url? -> http://localhost

This will gererate a WebdriverIO configuration file named `wdio.conf.js`. Finally we can create our first test, let's create a spec file named `hello.world.js` under `${PROJECT_HOME}/test/specs/hello.world.js`

```javascript
const assert = require('assert')

describe('Loading webpage', () => {

   it('validates website title', async() => {
       await browser.url('https://josdem.io/')
       const title = await browser.getTitle()
       assert.strictEqual(title, "josdem")
   })

})
```

You are good to execute it with: `npx wdio run wdio.conf.js`, and you should see this output:

```bash
"spec" Reporter:
------------------------------------------------------------------
[chrome 94.0.4606.81 linux #0-0] Running: chrome (v94.0.4606.81) on linux
[chrome 94.0.4606.81 linux #0-0] Session ID: 6c452fa2e8f3528b031a3f92b47620e3
[chrome 94.0.4606.81 linux #0-0]
[chrome 94.0.4606.81 linux #0-0] » /test/specs/helloWorld.js
[chrome 94.0.4606.81 linux #0-0] Loading webpage
[chrome 94.0.4606.81 linux #0-0]    ✓ validates website title
[chrome 94.0.4606.81 linux #0-0]
[chrome 94.0.4606.81 linux #0-0] 1 passing (1.1s)
```

To browse the code go [here](https://github.com/josdem/webdriverio-getting-started), to download the project:

```bash
git clone git@github.com:josdem/webdriverio-getting-started.git
```

[Return to the main article](/techtalk/ux)
