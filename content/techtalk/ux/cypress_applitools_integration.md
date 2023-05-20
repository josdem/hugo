+++
title =  "Cypress Applitools Integration"
description = "Cypress_applitools_integration"
date = "2023-05-20T19:27:20-04:00"
tags = ["josdem", "techtalks","programming","technology", "automation", "cypress", "applitools"]
categories = ["techtalk", "code", "ui", "automation", "visual validation"]
+++


In the same way, we open our eyes and analyze if an image looks good; [Applitools](https://applitools.com/) uses [Visual AI](https://www.syte.ai/blog/visual-ai/what-is-visual-ai/) to detect changes in our website so we can identify UI defects or validate expected new functionality. In this technical post, we will go over Applitools technology and how it can save us a ton of time in visual component validations. NOTE: If you need to know how to setup [Cypress](https://www.cypress.io/), please refer my previous post: [Cypress Getting Started](/techtalk/ux/cypress_getting_started/). Then add Applitools dependency:

```bash
npm install @applitools/eyes-cypress
npx eyes-setup
```

Now let’s create an applitools.config.js under our `${PROJECT_HOME}` directory

```javascript
module.exports = {
  testConcurrency: 5,
  batchName: "Cypress Vetlog",
  browser: [
    { width: 1280, height: 1024, name: "chrome" },
    { width: 1600, height: 1200, name: "firefox" },
    { width: 1280, height: 1024, name: "edge" },
    { width: 1024, height: 768, name: "safari" },
    { deviceName: "Pixel 5", screenOrientation: "portrait" },
    { deviceName: "iPhone 11", screenOrientation: "portrait" },
  ],
}
```

This configuration allows us to run visual tests across multiple browsers and devices simultaneously, saving time in execution.

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
describe("Loading home page", () => {
  before(function () {
    cy.fixture("test").then((data) => {
      this.data = data
    })
  })

  beforeEach(() => {
    cy.eyesOpen({
      appName: "Vetlog Application",
      testName: Cypress.currentTest.title,
    })
  })

  it("validates page title", function () {
    cy.visit(this.data.vetlogUrl)
    cy.eyesCheckWindow({
      tag: "Home Page",
      target: "window",
      fully: false,
    })
    cy.title().should("eq", this.data.expectedTitle)
  })

  afterEach(() => {
    cy.eyesClose()
  })
})
````
To run the project:
```bash
npx cypress run --browser chrome
```
Go to the [Applitools eyes dashboard](https://eyes.applitools.com/app/test-results/) and you should be able to see your batch images.

<img src="/img/techtalks/ux/vetlog_applitools.png">

To browse the code go [here](https://github.com/josdem/cypress-workshop), to download the project:

```bash
git clone git@github.com:josdem/cypress-workshop.git
```

[Return to the main article](/techtalk/ux)