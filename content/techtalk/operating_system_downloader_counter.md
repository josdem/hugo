+++
date = "2015-08-29T18:15:08-05:00"
draft = true
title = "Grails Example Project (Downloader stat)"

+++
Simple project with Grails

## Project user story
As application owner I want to deliver my application installer so the client can download it.

## Acceptance Criteria

* I need to know how many downloads I have by installer
* Installers are Linux, Ubuntu, Mac and Windows
* I need to know what is the client IP address

As first step, I’m going to create a project as follow:

```bash
grails create-app operating-system-downloader-stat
```

Next, I’m going to create a Domain as follow:

```bash
grails create-domain-class com.tim.Downloader
```

This domain Downloader will store download stats by operating system, it looks like this:

```groovy
package com.tim

 class Downloader {
   Date dateCreated
   String address
   InstallerType type

   static constraints = {
     address blank:false,size:5..255
   }
}
```

## About Domain

* Grails will create an "downloader" table
* If you define an property "dateCreated" it will set current date at creating new instances
* Grails will validate "address" is not empty and a size between 5 and 255
* InstallerType is an enum and I need to define it in src/groovy/ folder

## Creating enum
Create path src/groovy/com/tim Create InstallerType enum as follow:

```groovy
package com.tim

enum InstallerType {
 LINUX, MAC, UBUNTU, WINDOWS
}
```

## Creating controller
In order to create a controller we need to start grails application, type this:

```bash
grails
```

And then:

```bash
create-controller com.tim.DownloaderController
```

## Creating service
Services are transactional by default in Grails and is intended contains business logic, in order to create a service type:

```bash
grails
```

And then:

```bash
create-service com.tim.DownloaderService
```

## Unit testing
Now we are ready to write unit test and let the test guide us to the solution, we are going to start with DownloaderControllerSpec.groovy which is located in test/unit/com/tim

```groovy
package com.tim

import grails.test.mixin.TestFor
import spock.lang.Specification

@TestFor(DownloaderController)
class DownloaderControllerSpec extends Specification {
  DownloaderService downloaderService = Mock(DownloaderService)
  String address = "127.0.0.1"

  def setup(){
    controller.downloaderService = downloaderService
  }

  void "should count ubuntu download"() {
  when:
    controller.downloadUbuntuVersion()
  then:
    1 * downloaderService.createUbuntuStat(address)
  }
}
```

## DownloaderControllerSpec facts

* DownloaderService is a mock, needs to be that way since we are trying to test DownloaderController
* The purpose "def setup()" method is execute code before any test is called
* We assigned downloaderService to the controller in line 11
* When we call controller.downloadUbuntuVersion() we expect to call downloaderService.createUbuntuStat() once

You can run unit tests by typing in your command line:

```bash
grails test-app :unit
```

Now is time to set business logic which is create a record any time someone download an ubuntu version, so let’s our DownloaderServiceSpec lead us to the light

```groovy
package com.tim

import grails.test.mixin.TestFor
import spock.lang.Specification

@TestFor(DownloaderService)
class DownloaderControllerSpec extends Specification {

  void "should create a ubuntu download stat"() {
  when:
    def downloader = service.createUbuntuStat("127.0.0.1")
  then:
    downloader.address == "127.0.0.1"
    downloader.type == InstallerType.UBUNTU
  }
}
```

## DownloaderServiceSpec facts

* When we call service.createUbuntuStat("127.0.0.1") we are expecting that service returns an downloader object
* Then we verify that object contains "127.0.0.1" as address and UBUNTU as InstallerType

That’s it, the first part of the story is complete, we know what is the client IP address and when an Ubuntu package is downloaded. Next step is to deliver my Ubuntu package as downloader file, let’s return to our DownloaderControllerSpec

```groovy
package com.tim

import grails.test.mixin.TestFor
import spock.lang.Specification

@TestFor(DownloaderController)
class DownloaderControllerSpec extends Specification {
  DownloaderService downloaderService = Mock(DownloaderService)
  String address = "127.0.0.1"

  def setup(){
    controller.downloaderService = downloaderService
  }

  void "should count ubuntu download"() {
  when:
    controller.downloadUbuntuVersion()
  then:
    1 * downloaderService.createUbuntuStat(address)
    response.contentType == "application/octet-stream"
    response.getHeader("Content-disposition") =="attachment;filename=JMetadata.deb"
  }
}
```

## DownloaderServiceSpec modifications

* Now we are expecting that ContentType is application/octet-stream
* The intended purpose is to be saved to disk as "arbitrary binary data"
* We are expecting that Content-disposition is an attachment named JMetadata.deb

Now is time to see the DownloadController and DownloadService implementations

## DownloaderController

```groovy
package com.tim

class DownloaderController {
  def DownloaderService downloaderService

  def downloadUbuntuVersion(){
    downloaderService.createUbuntuStat(request.getRemoteAddr())

    def file = new File("/home/josdem/.jmetadata/JMetadata.deb")
    response.setContentType("application/octet-stream")
    response.setHeader("Content-disposition","attachment;filename=${file.getName()}")
    response.outputStream << file.newInputStream()
  }
}
```

## DownloaderService

```groovy
package com.tim

import grails.transaction.Transactional

@Transactional
class DownloaderService {

  def Downloader createUbuntuStat(String address){
    def downloader = new Downloader()
    downloader.address = address
    downloader.type = InstallerType.UBUNTU
    downloader.save()
  }
}
```

[Return to the main article](/techtalk/grails)

