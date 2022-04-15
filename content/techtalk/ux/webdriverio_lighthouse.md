+++
title =  "WebdriverIO Lighthouse"
description = "Implementing Lighthouse in WebdriverIO"
date = "2022-04-15T08:13:26-04:00"
tags = ["josdem", "techtalks","programming","technology", "webdriverio"]
categories = ["techtalk", "code", "nodejs", "webdriverio"]
+++

[Lighthouse](https://developers.google.com/web/tools/lighthouse) It is a performance tool for measuring and monitoring a website. You can run Lighthouse as Chrome extension, from command line, or as a Node module. Coming with [WebdriverIO](https://webdriver.io/) you can use it with devtools service. In this technical post we will go through the process to configure and use Lighthouse module as a service to run chrome devtools commands in a spec test with [Mocka Framework](https://mochajs.org/). **NOTE:** If you need to know how to setup a WebdriverIO, please refer my previous post: [WebdriverIO Getting Started](/techtalk/ux/webdriverio_getting_started/). Then add devtools dependency:

```bash
npm i --save-dev @wdio/devtools-service
```

Then edit `wdio.conf.js` and add `devtools` service to the webdriverIO configuration.

```javascript
// Test runner services
// Services take over a specific job you don't want to take care of. They enhance
// your test setup with almost no effort. Unlike plugins, they don't add new
// commands. Instead, they hook themselves up into the test process.
services: ["chromedriver", "devtools"],
```

Now let's create a test scenario where we can get a Lighthouse performance metrics.

```javascript
const assert = require("assert")
const properties = require(`../properties/${process.env.NODE_ENV}.properties`)

describe("Loading website", () => {
  before(() => {
    browser.enablePerformanceAudits()
  })

  it("Getting Lighthouse score", async () => {
    await browser.url(properties.url)
    await browser.getPageWeight()
    let metrics = await browser.getMetrics()
    let score = await browser.getPerformanceScore()
    console.log("metrics:", metrics)
    console.log("score: ", score)
    assert.ok(score > 0.5)
  })

  after(() => {
    browser.disablePerformanceAudits()
  })
})
```

You are good to execute this project with: `npx wdio run wdio.conf.js`, and you should see those metrics in the console. To browse the code go [here](https://github.com/josdem/webdriverio-workshop), to download the project:

```bash
git clone git@github.com:josdem/webdriverio-workshop.git
```

[Return to the main article](/techtalk/ux)
