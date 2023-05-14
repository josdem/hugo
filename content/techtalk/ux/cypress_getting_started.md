+++
title =  "Cypress Getting Started"
description = "Cypress_getting_started"
date = "2023-05-14T10:42:51-04:00"
tags = ["josdem", "techtalks","programming","technology", "automation", "cypress"]
categories = ["techtalk", "code", "ui", "automation"]
+++

[Cypress](https://www.cypress.io/) is a next-generation front-end testing tool built for modern web applications; it runs on Chrome, Firefox, Edge, and Electron. Cypress is most often compared to Selenium; however, Cypress is both fundamentally and architecturally different. Cypress is faster in execution plus facilitates UI selectors interaction and validations. Keep in mind that Cypress is a JavaScript only tool. Let's start by installing NodeJS on our computer, I highly recommend using [NVM](https://github.com/nvm-sh/nvm) so you can manage different versions. Once installed, let's create a new directory named `cypress-getting-started` and execute this command inside that new directory:

```bash
npm init -y
```

This will create a new `package.json` file and it should looks like this:

```javascript
{
  "name": "cypress-workshop",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```
Next step is to use `npm` command to install Cypress.

```bash
npm install cypress
```
Now we want to generate some Cypress structure in our project; in order to do that, let's run Cypress in an interactive mode with this command.
```bash
npx cypress open
```
Select default options and create a new spec which is a test file named it as:
```bash
cypress\e2e\home.spec.cy.js
```
Open that file and add this code:
```javascript
describe("Loading home page", () => {
  
    it("validates page title", function () {
      cy.visit("https://vetlog.org/")
      cy.title().should("eq", "Vetlog")
    })
    
})
```
That's it; we open an URL [https://vetlog.org/](https://vetlog.org/) and validating the page title is equal to "Vetlog", pretty straight forward; however, following good practices, we need to externalize this URL and expected titles; Cypress provides us with a feature to drive the data from external sources called fixtures, which are located in `${PROJECT_HOME}/cypress/fixtures`. Let's create one named: `test.json` with this content:
```javascript
{
  "vetlogUrl": "http://vetlog.org",
  "expectedTitle": "Vetlog"
}
```
We can use `cy.fixture()` in the `before()` structure to read all data from the fixture file "test.json" to the local object "this".
 ```javascript
 describe("Loading home page", () => {

  before(function () {
    cy.fixture("test").then((data) => {
      this.data = data
    })
  })
  
  it("validates page title", function () {
    cy.visit("https://vetlog.org/")
    cy.title().should("eq", this.data.expectedTitle)
  })

})
```
To run the project:
```bash
npx cypress run --browser chrome
```
To browse the code go [here](https://github.com/josdem/cypress-getting-started), to download the project:

```bash
git clone git@github.com:josdem/webdriverio-getting-started.git
```

[Return to the main article](/techtalk/ux)

