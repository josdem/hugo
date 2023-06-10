+++
title =  "Playwright Getting Started"
description = "Playwright was created to accommodate the needs of end-to-end testing. It supports all modern rendering engines including Chromium, WebKit and Firefox."
date = "2023-06-10T13:34:08-04:00"
tags = ["josdem", "techtalks","programming","technology", "automation", "playwright"]
categories = ["techtalk", "code", "ui", "automation"]
+++

[Playwright](https://playwright.dev/) was created to accommodate the needs of end-to-end testing. It supports all modern rendering engines, including Chromium, WebKit, and Firefox. It has an incredible architecture using [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) connection protocol, which means more efficient communication between test commands and actions in your browsers. Another great advantage is that it runs all your test in parallel and generates an excelent build-in report. Let's start by installing NodeJS on our computer, I highly recommend using [NVM](https://github.com/nvm-sh/nvm) so you can manage different versions. Once installed, let's create a new directory named `playwright-getting-started` and execute this command inside that new directory:

```bash
npm init playwright@latest
```

When, select these options:
- JavaScript
- Where to put your end-to-end tests? => Test
- Add a GitHub Actions workflow? => y
- Install Playwright browsers (can be done manually via 'npx playwright install')? => Y

Then, you should see this output:

```bash
And check out the following files:
  - .\tests\example.spec.js - Example end-to-end test
  - .\tests-examples\demo-todo-app.spec.js - Demo Todo App end-to-end tests
  - .\playwright.config.js - Playwright Test configuration

Visit https://playwright.dev/docs/intro for more information. ✨

Happy hacking! 🎭
```

That's right! Playwright supports [GitHub Actions](https://github.com/features/actions) by default; how cool is that? In the `${PROJECT_HOME}/tests/` directory create a file named: `home.spec.js` with this content:

```javascript
const { test, expect } = require("@playwright/test")

test("should validate page title", async ({ page }) => {
  await page.goto("https://vetlog.org/")
  await expect(page).toHaveTitle("Vetlog")
})
```

To run our test, type this command:

```bash
npx playwright test
```

You should get this output:

```bash
Running 3 tests using 3 workers
  3 passed (6.2s)
```
As you can see, our test was executed in three browsers: Chromium, Firefox, and WebKit(Safari). We do not want to hardcode values, therefore let's create a properties file named: `test.properties.js` under `${PROJECT_HOME}/properties/` directory with this content:

```javascript
module.exports = {
  url: "https://vetlog.org/",
  title: "Vetlog",
}
```

So that, we can insert these values in our test

```javascript
const { test, expect } = require("@playwright/test")
const properties = require("../properties/test.properties")

test("should validate page title", async ({ page }) => {
  await page.goto(properties.url)
  await expect(page).toHaveTitle(properties.title)
})
```

To open last HTML report run:

```bash
npx playwright show-report
```

To browse the code go [here](https://github.com/josdem/playwright-getting-started), to download the project:

```bash
git clone git@github.com:josdem/playwright-workshop.git
```

[Return to the main article](/techtalk/ux)