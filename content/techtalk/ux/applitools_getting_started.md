+++
title =  "Applitools Getting Started"
description = "Applitools_getting_started"
date = "2021-11-20T11:22:39-05:00"
tags = ["josdem", "techtalks","programming","technology","WebdriverIO", "Applitools"]
categories = ["techtalk", "code", "NodeJS", "JavaScript","WebdriverIO", "Applitools"]
+++

In the same way we open our eyes and analyze if an image looks good, [Applitools](https://applitools.com/) uses [Visual AI](https://www.syte.ai/blog/visual-ai/what-is-visual-ai/) to detect changes in our website so we can identify UI defects or validate expected new functionality. In this technical post we will go over Applitools technology and how it can save us to of time in visual components validations. **NOTE:** If you need to know how to setup a [WebdriverIO](https://webdriver.io/), please refer my previous post: [Spring Boot](/techtalk/ux/webdriverio_getting_started/). Then add Applitools dependency:

```bash
npm i --save-dev @applitools/eyes-webdriverio
```

Now let's create a `navigate.spec.js` under `test/specs/` directory with this content

```bash
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

Let's now analyze every section, on `before` method we are setting up all required Applitools setup, on `beforeEach` we are setting up our test information such as application and test name. it "validates website title" we are opening our taget web page and taking an screenshot with `await applitools.checkWindowEyes("home page")`

To browse the code go [here](https://github.com/josdem/applitools-workshop), to download the project:

```bash
git clone git@github.com:josdem/applitools-workshop.git
```

[Return to the main article](/techtalk/ux)