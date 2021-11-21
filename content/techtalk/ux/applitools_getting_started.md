+++
title =  "Applitools Getting Started"
description = "Applitools_getting_started"
date = "2021-11-20T11:22:39-05:00"
tags = ["josdem", "techtalks","programming","technology","WebdriverIO", "Applitools"]
categories = ["techtalk", "code", "NodeJS", "JavaScript","WebdriverIO", "Applitools"]
+++

In the same way we open our eyes and analyze if an image looks good, [Applitools](https://applitools.com/) uses [Visual AI](https://www.syte.ai/blog/visual-ai/what-is-visual-ai/) to detect changes in our website so we can identify UI defects or validate expected new functionality. In this technical post we will go over Applitools technology and how it can save us to of time in visual components validations. **NOTE:** If you need to know how to setup a [WebdriverIO](https://webdriver.io/), please refer my previous post: [WebdriverIO Getting Started](/techtalk/ux/webdriverio_getting_started/). Then add Applitools dependency:

```bash
npm i --save-dev @applitools/eyes-webdriverio
```

Now let's create a `navigate.spec.js` under `test/specs/` directory with this content

```javascript
const properties = require(`../properties/${process.env.NODE_ENV}.properties`)

import * as assert from "assert"
import * as applitools from "../utils/applitools.util"
import { HomePage } from "../pageobjects/home.page"

describe("Loading WebdriverIO webpage", () => {
  before("setting up Applitools configuration", async () => {
    await applitools.setUpConfiguration()
  })

  beforeEach("setting up test information", async function () {
    const appName = await this.test.parent.title
    const testName = await this.currentTest.title
    await applitools.setUpTest(appName, testName)
  })

  it("validates website title", async () => {
    const title = await HomePage.open()
    await applitools.checkWindowEyes("home page")
    assert.strictEqual(title, properties.title)
    await applitools.closeEyes()
  })

  afterEach("cleaning up test", async () => {
    await applitools.cleaning()
  })

  after("publishing results", async () => {
    await applitools.publishing()
  })
})
```

Let's analyze every section, on `before` method we are setting up all required Applitools setup, on `beforeEach` we are setting up our test information such as application and test name. it "validates website title" we are opening our taget web page and taking an screenshot with `await applitools.checkWindowEyes("home page")` on `afterEach` and `after` methods we are cleaning and publishing our results. Now is time to take a look to Applitools confuguration in detail:

```javascript
const setUpConfiguration = async () => {
  const runnerOptions = new RunnerOptions().testConcurrency(5)
  runner = new VisualGridRunner(runnerOptions)
  eyes = new Eyes(runner)

  configuration = eyes.getConfiguration()
  configuration.setApiKey(process.env.APPLITOOLS_API_KEY)
  configuration.setServerUrl(APPLITOOLS_SERVER)
  configuration.setBatch(new BatchInfo(BATCH_INFO))
  configuration.addBrowser(CHROME.width, CHROME.height, BrowserType.CHROME)
  configuration.addBrowser(FIREFOX.width, FIREFOX.height, BrowserType.FIREFOX)
  configuration.addBrowser(EDGE.width, EDGE.height, BrowserType.EDGE_CHROMIUM)
  configuration.addBrowser(SAFARI.width, SAFARI.height, BrowserType.SAFARI)
  configuration.addDeviceEmulation(DeviceName.iPhone_11, ScreenOrientation.PORTRAIT)
  configuration.addDeviceEmulation(DeviceName.Pixel_5, ScreenOrientation.PORTRAIT)
}
```

As you can see here we are setting up our Applitools key from the system properties, you might want to get your own key, here is the link to get it: [Create an account](https://auth.applitools.com/users/register). Here we are adding all browsers and devices we want to test our web page, yes Applitools will create an screenshot for every device / browser specified.

```javascript
const checkWindowEyes = async (screenshot) => {
  await eyes.check(screenshot, Target.window().layoutBreakpoints(BREAK_POINT_SIZE))
}
```

On `checkWindowEyes` we are taking an screenshot using `eyes.check` method. There are cases where a tested page change its layout and content based on viewport width, that is why is a good idea to use `layoutBreakpoints` to specify a break point size. For example if you know that your web page changes the layout at 700 pixels from desktop to mobile view then we can specify these values as the layout breakpoints.

```javascript
const closeEyes = async () => {
  await eyes.closeAsync()
}
```

On `closeEyes` we are closing asynchronously Applitools eyes, this is important since we are using WebdriverIO in an asynchronous implementation. Here is the complete Applitools utils code:

```javascript
const { VisualGridRunner, RunnerOptions, Eyes, Target, BatchInfo, BrowserType, DeviceName, ScreenOrientation } = require("@applitools/eyes-webdriverio")

const APPLITOOLS_SERVER = "https://eyes.applitools.com/"
const BATCH_INFO = "Ultrafast Batch"
const BREAK_POINT_SIZE = 700
const CHROME = {
  width: 800,
  height: 600,
}
const FIREFOX = {
  width: 800,
  height: 600,
}
const EDGE = {
  width: 800,
  height: 600,
}
const SAFARI = {
  width: 1024,
  height: 768,
}

let eyes
let runner
let configuration

const setUpConfiguration = async () => {
  const runnerOptions = new RunnerOptions().testConcurrency(5)
  runner = new VisualGridRunner(runnerOptions)
  eyes = new Eyes(runner)

  configuration = eyes.getConfiguration()
  configuration.setApiKey(process.env.APPLITOOLS_API_KEY)
  configuration.setServerUrl(APPLITOOLS_SERVER)
  configuration.setBatch(new BatchInfo(BATCH_INFO))
  configuration.addBrowser(CHROME.width, CHROME.height, BrowserType.CHROME)
  configuration.addBrowser(FIREFOX.width, FIREFOX.height, BrowserType.FIREFOX)
  configuration.addBrowser(EDGE.width, EDGE.height, BrowserType.EDGE_CHROMIUM)
  configuration.addBrowser(SAFARI.width, SAFARI.height, BrowserType.SAFARI)
  configuration.addDeviceEmulation(DeviceName.iPhone_11, ScreenOrientation.PORTRAIT)
  configuration.addDeviceEmulation(DeviceName.Pixel_5, ScreenOrientation.PORTRAIT)
}

const setUpTest = async (appName, testName) => {
  configuration.setAppName(appName)
  configuration.setTestName(testName)
  eyes.setConfiguration(configuration)
  await eyes.open(browser)
}

const checkWindowEyes = async (screenshot) => {
  await eyes.check(screenshot, Target.window().layoutBreakpoints(BREAK_POINT_SIZE))
}

const closeEyes = async () => {
  await eyes.closeAsync()
}

const cleaning = async () => {
  await eyes.abortAsync()
}

const publishing = async () => {
  const results = await runner.getAllTestResults(false)
  console.log(results)
}

module.exports = {
  setUpConfiguration,
  setUpTest,
  checkWindowEyes,
  closeEyes,
  cleaning,
  publishing,
}
``` 

You are good to execute it with: `npx wdio run wdio.conf.js`, and you should see this output:

<img src="/img/techtalks/ux/applitools.png">

To browse the code go [here](https://github.com/josdem/applitools-workshop), to download the project:

```bash
git clone git@github.com:josdem/applitools-workshop.git
```

[Return to the main article](/techtalk/ux)