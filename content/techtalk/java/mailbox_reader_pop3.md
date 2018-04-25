+++
title = "GMail Mailbox Reader with POP3"
date = "2017-05-18T16:29:30-05:00"
categories = ["techtalk","code","java"]
tags = ["josdem","techtalks","programming","technology","mailbox reader","pop3 client","java"]
description = "This time I will show you how to read a Gmail mailbox using POP3 protocol and Java Mail."
+++

This time I will show you how to read a Gmail mailbox using POP3 protocol and Java Mail. First you need to define your `build.gradle` as follow:

```groovy
def configurationDirectory = "${System.getProperty('user.home')}/.mailbox-reader"

buildscript {
  ext {
    springBootVersion = '1.5.1.RELEASE'
    cglibVersion = '3.2.4'
    javaMailVersion = '1.4'
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

Then we define a common interface to support different implementations

```groovy
package com.jos.dem.mailbox.reader.service

interface InboxReader {
  void setup()
  void read()
}
```

This is the POP3 mailbox reader implementation

```groovy
package com.jos.dem.mailbox.reader.service.impl

import org.springframework.stereotype.Service
import org.springframework.beans.factory.annotation.Value
import org.springframework.beans.factory.annotation.Autowired

import javax.mail.Folder
import javax.mail.Message
import javax.mail.Session
import javax.mail.Store
import javax.annotation.PostConstruct

import com.jos.dem.mailbox.reader.service.InboxReader

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@Service
class InboxReaderPop3 implements InboxReader {

  @Value('${username}')
  String username
  @Value('${password}')
  String password
  @Value('${server}')
  String server
  @Value('${port}')
  String port

  Properties properties = new Properties()
  Folder emailFolder
  Store store

  Logger log = LoggerFactory.getLogger(this.class)
	
  @PostConstruct
  void setup() {
	properties.put('mail.pop3.host', server)
	properties.put('mail.pop3.port', port)
	properties.put('mail.pop3.starttls.enable', 'true')
	Session emailSession = Session.getDefaultInstance(properties)
	Store store = emailSession.getStore('pop3s')
	store.connect(server, username, password)
	emailFolder = store.getFolder('INBOX')
	emailFolder.open(Folder.READ_ONLY)
	log.info "Inbox Type: ${emailFolder.getType()}"
  }

  void read(){
	Message[] messages = emailFolder.getMessages()
	log.info "Messages Length: ${messages.length}"
	for (int i = 0; i < messages.length; i++) {
	  Message message = messages[i]
	  log.info '--------------------------------'
	  log.info "Email Number ${(i + 1)}"
	  log.info "From: ${message.getFrom()[0]}" 
	  log.info "Subject: ${message.getSubject()}"
	}
    emailFolder.close(false)
  }
	
}
```

`javax.mail.Folder` has this types: 

* HOLDS_FOLDER = 1
* HOLDS_MESSAGES = 1
* READ_ONLY = 1
* READ_WRITE = 2

*Output*

```bash
Inbox Type: 1
Reading message
Messages Length: 1
--------------------------------
Email Number 1
From: =Jose Luis De la Cruz Morales= <joseluis.delacruz@gmail.com>
Subject: Hello POP3 mailbox reader implementation
```

**Configuration**

In your computer's home directory: ${home}, create a directory called: .mailbox-reader then inside create a file called application.properties with this content

```bash
username=user@gmail.com
password=secret
pop3.server=pop.gmail.com
pop3.port=995
imap.server=imap.gmail.com
imap.port=993
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
