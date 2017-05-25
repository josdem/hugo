+++
title = "Mailbox Reader Exchange"
categories = ["techtalk","code"]
tags = ["josdem","techtalks","programming","technology","mailbox reader","exchange client"]
date = "2017-05-25T17:52:58-05:00"

+++

This time I will show you how to read an Outlook Exchange mailbox using Exchange Web Services Java API [ews-java-api](https://github.com/OfficeDev/ews-java-api). First you need to define your `build.gradle` as follow:

```groovy
def configurationDirectory = "${System.getProperty('user.home')}/.mailboxi-reader"

buildscript {
  ext {
    springBootVersion = '1.5.1.RELEASE'
    cglibVersion = '3.2.4'
    javaMailVersion = '1.4'
    ewsJavaApiVersion = '2.0'
  }
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  }
}

apply plugin: "groovy"
apply plugin: "application"
apply plugin: 'org.springframework.boot'

jar {
  baseName = 'mailbox-reader'
  version = '0.0.1-SNAPSHOT'
}

sourceCompatibility = 1.8

repositories {
  mavenCentral()
}

dependencies {
  compile 'org.springframework.boot:spring-boot-starter'
  compile 'org.codehaus.groovy:groovy'
  compile "javax.mail:mail:$javaMailVersion"
  compile "com.microsoft.ews-java-api:ews-java-api:$ewsJavaApiVersion"
  testCompile 'org.springframework.boot:spring-boot-starter-test'
  testCompile 'org.spockframework:spock-spring'
  testCompile "cglib:cglib-nodep:$cglibVersion"
}

task settingEnvironment(type:Copy) {
  from configurationDirectory
  into 'src/main/resources'
  include "application.properties"
}

processResources.dependsOn "settingEnvironment"
```

This is the Exchange mailbox reader implementation

```groovy
package com.jos.dem.mailbox.reader.service.impl

import org.springframework.stereotype.Service
import org.springframework.beans.factory.annotation.Value
import org.springframework.beans.factory.annotation.Autowired

import microsoft.exchange.webservices.data.core.ExchangeService
import microsoft.exchange.webservices.data.core.PropertySet
import microsoft.exchange.webservices.data.core.service.item.Item
import microsoft.exchange.webservices.data.core.service.folder.Folder
import microsoft.exchange.webservices.data.core.service.item.EmailMessage
import microsoft.exchange.webservices.data.credential.WebCredentials
import microsoft.exchange.webservices.data.credential.ExchangeCredentials
import microsoft.exchange.webservices.data.misc.ImpersonatedUserId
import microsoft.exchange.webservices.data.search.ItemView
import microsoft.exchange.webservices.data.search.FindItemsResults
import microsoft.exchange.webservices.data.property.complex.Mailbox
import microsoft.exchange.webservices.data.property.complex.FolderId
import microsoft.exchange.webservices.data.core.enumeration.misc.ConnectingIdType
import microsoft.exchange.webservices.data.autodiscover.IAutodiscoverRedirectionUrl
import microsoft.exchange.webservices.data.core.enumeration.property.WellKnownFolderName

import javax.annotation.PostConstruct

import com.jos.dem.mailbox.reader.service.InboxReader

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@Service
class InboxReaderExchange implements InboxReader {

  private static final Integer MAX_ITEMS=50

  @Value('${ews.username}')
  String username
  @Value('${ews.password}')
  String password
  @Value('${ews.server}')
  String server
  @Value('${ews.protocol}')
  String protocol

  ExchangeService service = new ExchangeService()

  Logger log = LoggerFactory.getLogger(this.class)

  @PostConstruct
  void setup(){
    service.setUrl(new URI(server))
    ExchangeCredentials credentials = new WebCredentials(username, password)
    service.setCredentials(credentials)
    service.autodiscoverUrl(username,  new RedirectionUrlCallback(protocol))
  }

  void read(){
    Folder folder = Folder.bind(service, WellKnownFolderName.Inbox)
    FindItemsResults<Item> results = service.findItems(folder.getId(), new ItemView(MAX_ITEMS))
    for (Item item : results) {
      EmailMessage emailMessage = EmailMessage.bind(service, item.getId())
      log.info("Sender: ${emailMessage.getSender()}")
      log.info("Subject: ${emailMessage.getSubject()}")
    }
  }

}

class RedirectionUrlCallback implements IAutodiscoverRedirectionUrl {

  String protocol

  RedirectionUrlCallback(String protocol){
    this.protocol = protocol
  }
  public boolean autodiscoverRedirectionUrlValidationCallback(
    String redirectionUrl) {
    return redirectionUrl.toLowerCase().startsWith(protocol)
  }

}
```

*Output*

```bash
Started MailboxReaderApplication in 9.162 seconds (JVM running for 10.01)
Reading message
Sender: TheDailyInc <noreply@outlook.com>
Subject: Team at Cannes, VIA progress report, Brand Central, and Houston groundbreaking
Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@763d9750
```

If the domain that the user inputs as their email address contains a CNAME that redirects the user this Exception is thrown:

```bash
microsoft.exchange.webservices.data.autodiscover.exception.AutodiscoverLocalException: Autodiscover blocked a potentially insecure redirection to URL. To allow Autodiscover to follow the redirection, use the AutodiscoverUrl(string, AutodiscoverRedirectionUrlValidationCallback) overload.
```

When this happens, instead of failing, the user can be prompted to accept the redirection or not. That functionality needs to be implemented inside the autodiscoverRedirectionUrlValidationCallback method.


**Configuration**

In your computer's home directory: ${home}, create a directory called: .mailbox-reader then inside create a file called application.properties with this content

```bash
username=user@gmail.com
password=secret
pop3.server=pop.gmail.com
pop3.port=995
imap.server=imap.gmail.com
imap.port=993
ews.username=user@outlook.com
ews.password=secret
ews.server=https://exchange/EWS/Exchange.asmx
ews.protocol=https://
```

**Build**

```bash
gradle build
```

**Run**

```bash
 java -jar build/libs/mailbox-reader-0.0.1-SNAPSHOT.jar
```

To download the code:

```bash
git clone https://github.com/josdem/java-topics.git
cd mailbox-reader
```


[Return to the main article](/techtalk/java)
