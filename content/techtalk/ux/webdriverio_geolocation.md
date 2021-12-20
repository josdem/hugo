+++
title =  "Webdriverio Geolocation"
description = "Webdriverio_geolocation"
date = "2021-12-20T13:17:33-06:00"
tags = ["josdem", "techtalks","programming","technology","WebdriverIO"]
categories = ["techtalk", "code", "NodeJS", "JavaScript","WebdriverIO"]
+++


In this technical post we will go over the process to mimic user's geolocation using [WebdriverIO](https://webdriver.io/) along with [Mocha Framework](https://mochajs.org/). **NOTE:** If you need to know how to setup a WebdriverIO, please refer my previous post: [WebdriverIO Getting Started](/techtalk/ux/webdriverio_getting_started/). Then add devtools dependency:

```bash
npm i --save-dev @wdio/devtools-service
```

Now let's create a test scenario where we modify our location to be in Ann Arbor Michigan, United States.

```javascript
describe("Checking locations", () => {
  it("goes to Ann Arbor Michigan", async () => {
    await LocationPage.open()
    await browser.cdp("Emulation", "setGeolocationOverride", {
      latitude: 42.3173603,
      longitude: -83.6826172,
      accuracy: 1,
    })
})
```

On LocationPage open method we are browsing this URL: https://www.where-am-i.net/ so we can get our current location. Another important method is `cbp` which is a custom command added to the browser that allows to call emulation `setGeolocationOverride` from devtools protocol, if you want to know more about this method please go [here](https://chromedevtools.github.io/devtools-protocol/tot/Emulation/#method-setGeolocationOverride)

```javascript
class LocationPage {

  async open() {
    await browser.url("https://www.where-am-i.net/")
  }
}

module.exports = new LocationPage()
```

<img src="/img/techtalks/ux/ann_arbor.png">

Let's move from United States Michigan to Mexico Guadalajara.

```javascript
describe("Checking locations", () => {
  it("goes from Ann Arbor to Guadalajara to Amsterdam", async () => {
    await LocationPage.open()
    await browser.cdp("Emulation", "setGeolocationOverride", {
      latitude: 42.3173603,
      longitude: -83.6826172,
      accuracy: 1,
    })

    await LocationPage.clickOnLocationButton()
    await browser.pause(3000)
    await browser.cdp("Emulation", "setGeolocationOverride", {
      latitude: 20.6743943,
      longitude: -103.3874128,
      accuracy: 1,
    })
})
```

On `LocationPage.clickOnLocationButton()` we are performing a click on location button so we can update our location on the map.

```javascript
const properties = require(`../properties/test.properties`)

class LocationPage {
  get locationButton() {
    return $("[id=btnMyLocation]")
  }

  async clickOnLocationButton() {
    const button = await this.locationButton
    await expect(button).toBeExisting()
    await button.click()
  }

  async open() {
    await browser.url(properties.url)
  }
}

module.exports = new LocationPage()
```

<img src="/img/techtalks/ux/guadalajara.png">


And finally let's move from Mexico to Netherlands, also here we are externalizing our values in a properties file, so we can avoid hard coding.

```javascript
const properties = require(`../properties/test.properties`)
const LocationPage = require("../pageobjects/location.page")

describe("Checking locations", () => {
  it("goes from Ann Arbor to Guadalajara to Amsterdam", async () => {
    await LocationPage.open()
    await browser.cdp("Emulation", "setGeolocationOverride", {
      latitude: properties.UNITED_STATES.latitude,
      longitude: properties.UNITED_STATES.longitude,
      accuracy: properties.accuracy,
    })

    await LocationPage.clickOnLocationButton()
    await browser.pause(properties.waitingTime)
    await browser.cdp("Emulation", "setGeolocationOverride", {
      latitude: properties.MEXICO.latitude,
      longitude: properties.MEXICO.longitude,
      accuracy: properties.accuracy,
    })

    await LocationPage.clickOnLocationButton()
    await browser.pause(properties.waitingTime)
    await browser.cdp("Emulation", "setGeolocationOverride", {
      latitude: properties.NETHERLANDS.latitude,
      longitude: properties.NETHERLANDS.longitude,
      accuracy: properties.accuracy,
    })
    await LocationPage.clickOnLocationButton()
    await browser.pause(properties.waitingTime)
  })
})
```

<img src="/img/techtalks/ux/amsterdam.png">

You are good to execute it with: `npx wdio run wdio.conf.js`, and you should see those geolocations in the map. To browse the code go [here](https://github.com/josdem/webdriverio-workshop), to download the project:

```bash
git clone git@github.com:josdem/webdriverio-workshop.git
```

[Return to the main article](/techtalk/ux)