+++
title =  "Playwright Applitools Integration"
description = "Playwright Applitools Integration"
date = "2023-06-13T08:41:26-04:00"
tags = ["josdem", "techtalks","programming","technology", "automation", "playwright", "applitools"]
categories = ["techtalk", "code", "ui", "automation", "visual validation"]
+++

In the same way, we open our eyes and analyze if an image looks good; [Applitools](https://applitools.com/) uses [Visual AI](https://www.syte.ai/blog/visual-ai/what-is-visual-ai/) to detect changes in our website so we can identify UI defects or validate expected new functionality. In this technical post, we will go over Applitools technology and how it can save us a ton of time in visual component validations. NOTE: If you need to know how to setup [Playwright](https://playwright.dev/), please refer my previous post: [Playwright Getting Started](/techtalk/ux/cypress_getting_started/). Then add Applitools dependency:

```bash
npm install @applitools/eyes-playwright --save-dev
```

Now let’s create an applitools.util.ts under our `${PROJECT_HOME}/utils` directory and add this content:

```javascript
import { BatchInfo, Configuration, VisualGridRunner, BrowserType, DeviceName, ScreenOrientation, Eyes, Target } from "@applitools/eyes-playwright"
import { Page } from "@playwright/test"

const CHROME = {
  width: 1280,
  height: 768,
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

let configuration
let runner
let eyes

export function setUpConfiguration(batchName: string){
  runner = new VisualGridRunner({ testConcurrency: 5 })
  configuration = new Configuration()
  configuration.setBatch(new BatchInfo(batchName))
  configuration.addBrowser(CHROME.width, CHROME.height, BrowserType.CHROME)
  configuration.addBrowser(FIREFOX.width, FIREFOX.height, BrowserType.FIREFOX)
  configuration.addBrowser(EDGE.width, EDGE.height, BrowserType.EDGE_CHROMIUM)
  configuration.addBrowser(SAFARI.width, SAFARI.height, BrowserType.SAFARI)
  configuration.addDeviceEmulation(DeviceName.iPhone_11, ScreenOrientation.PORTRAIT)
  configuration.addDeviceEmulation(DeviceName.Pixel_5, ScreenOrientation.PORTRAIT)
}

export async function setUpTest(page: Page, appName: string, testName: string){
  eyes = new Eyes(runner, configuration)
  await eyes.open(page, appName, testName)
}

export async function checkWindowEyes(screenshot: string){
  await eyes.check(screenshot, Target.window().layout())
}

export async function closeEyes(){
  await eyes.close()
}

export async function cleaning(){
  const results = await runner.getAllTestResults()
  console.log("Visual test results", results)
}

export default { setUpConfiguration, setUpTest, checkWindowEyes, closeEyes, cleaning }
```

On `checkWindowEyes` we are capturing a screenshot from the browser using `eyes.check` method, we can also capture from any region, UI element, or even entire window, you can know more about it [here](https://applitools.com/docs/method-eyes-check-seleniumide-command.html), and in our `setUpTest` we are initializing Applitools eyes with page, application name, and test information.

## Running tests

Before running the visual test, you must find your [Applitools API key](https://applitools.com/tutorials/guides/getting-started/registering-an-account#retrieving-your-api-key) and set it as an environment variable named `APPLITOOLS_API_KEY`

Linux / Mac
```bash
export APPLITOOLS_API_KEY=${YOUR_APPLITOOLS_API_KEY}
```

Windows:
```bash
$Env:APPLITOOLS_API_KEY="YOUR_APPLITOOLS_API_KEY"
```

This is how our home page looks like after integrating Applitools.

```javascript
import { test, expect } from "@playwright/test"
import { Constants } from "../properties/test.properties"
import applitools from "../utils/applitools.util"

test.beforeAll(async () => {
  applitools.setUpConfiguration(Constants.BATCH_NAME)
})

test.beforeEach(async ({ page }) => {
  await applitools.setUpTest(page, Constants.APP, test.info().title)
})

test("should validate page title", async ({ page }) => {
  await page.goto(Constants.URL)
  await expect(page).toHaveTitle(Constants.TITLE)
  await applitools.checkWindowEyes("Home Page")
})

test.afterEach(async () => {
  await applitools.closeEyes()
})

test.afterAll(async () => {
  await applitools.cleaning()
})
```

To run the project:

```bash
npx playwright test --project chromium
```

Go to the [Applitools eyes dashboard](https://eyes.applitools.com/app/test-results/) and you should be able to see your batch images.

<img src="/img/techtalks/ux/vetlog_playwright_applitools.png">

To browse the code go [here](https://github.com/josdem/playwright-applitools), to download the project:

```bash
git clone git@github.com:josdem/playwright-applitools.git
```

To go to javascript version:

```bash
git fetch
git checkout javascript
```

To go to typescript version:

```bash
git fetch
git checkout typescript
```

[Return to the main article](/techtalk/ux)
