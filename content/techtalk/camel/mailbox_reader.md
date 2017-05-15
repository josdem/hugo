+++
date = "2017-04-27T14:54:50-05:00"
title = "Mailbox Reader"
categories = ["techtalk","code"]
tags = ["josdem","techtalks","programming","technology","Gmail reader", "IMAP reader"]

+++


## Mail Component

The mail component provides access to Email via Spring's Mail support and the underlying JavaMail system. Add Gradle dependency in `gradle.properties` with this line.

```groovy
compile 'org.apache.camel:camel-mail:2.17.0'
```

In this sample we poll a mailbox using a Gmail account. Google mail requires you to enable IMAP access, this is done by logging into your account and set IMAP enabled (Settings > Forwarding and POP/IMAP)

```groovy
from("imaps://imap.gmail.com?username=josdem@email.com&password=password"
					+ "&delete=false&unseen=true&consumer.delay=60000")
```

The preceding route polls the Google mail inbox for new mails once every minute and logs the received messages to the newmail logger category.

```groovy
package com.jos.dem.mailbox.reader

import static java.nio.file.StandardCopyOption.REPLACE_EXISTING

import org.springframework.stereotype.Component

import org.apache.camel.CamelContext
import org.apache.camel.Processor
import org.apache.camel.Exchange
import org.apache.camel.Message
import org.apache.camel.impl.DefaultCamelContext
import org.apache.camel.builder.RouteBuilder

import javax.annotation.PostConstruct
import javax.activation.DataHandler
import javax.mail.internet.MimeMultipart
import javax.mail.internet.MimeMessage
import javax.mail.BodyPart

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@Component
class InboxReader {

	CamelContext context

  Logger log = LoggerFactory.getLogger(this.class)

	@PostConstruct
	void setup(){
		context = new DefaultCamelContext()
		context.addRoutes(new RouteBuilder(){
			void configure(){
				from("imaps://imap.gmail.com?username=josdem@email.com&password=password"
					+ "&delete=false&unseen=true&consumer.delay=60000")
				.process(new Processor() {
					@Override
					public void process(Exchange exchange) throws Exception {
						Message message = exchange.getIn()
						log.info "exchange: ${exchange.dump()}"
						log.info "message: ${message.dump()}"
						if(message.getBody() instanceof MimeMultipart){
							MimeMultipart mimeMultipart = message.getBody()
							log.info "mimeMultipart: ${mimeMultipart.dump()}"
						} else {
							String body = message.getBody()
							log.info "body: ${body}"
						}
					}	
					})
				.to("log:newmail");
			}
			})
	}

	def start(){
		context.start()
	}

	def stop(){
		context.stop()
	}
	
}
```

In the previous class I am creating a Apache Camel component with a route to read Gmail mailbox and process it using `exchange.getIn()` we got the message and within a body which can be `multipart/mixed` or `text/plain` as content type.

This is the launcher to start and stop the Apache Camel component:

```groovy
package com.jos.dem.mailbox.reader

import org.springframework.stereotype.Component
import org.springframework.beans.factory.annotation.Autowired

import com.jos.dem.mailbox.reader.InboxReader

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@Component
class Launcher{

  @Autowired
  InboxReader inboxReader

  Logger log = LoggerFactory.getLogger(this.class)

  void start(){
    log.info 'Reading message'
    inboxReader.start()
    Thread.sleep(60000)
    inboxReader.stop()
  }

}
```

See the code below to read an email attachment and process it.

```groovy
package com.hp.mailbox.reader

import static java.nio.file.StandardCopyOption.REPLACE_EXISTING

import java.nio.file.Path
import java.nio.file.Paths
import java.nio.file.Files

import org.springframework.stereotype.Component

import org.apache.camel.CamelContext
import org.apache.camel.Processor
import org.apache.camel.Exchange
import org.apache.camel.Message
import org.apache.camel.impl.DefaultCamelContext
import org.apache.camel.builder.RouteBuilder

import javax.annotation.PostConstruct
import javax.activation.DataHandler
import javax.mail.internet.MimeMultipart
import javax.mail.internet.MimeMessage
import javax.mail.BodyPart

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@Component
class InboxReader {

  CamelContext context

  Logger log = LoggerFactory.getLogger(this.class)

  @PostConstruct
  void setup(){
    context = new DefaultCamelContext()
    context.addRoutes(new RouteBuilder(){
      void configure(){
        from("imaps://imap.gmail.com?username=contact@josdem.io&password=password"
          + "&delete=false&unseen=true&consumer.delay=60000")
        .process(new Processor() {
          @Override
          public void process(Exchange exchange) throws Exception {
            Message message = exchange.getIn()
            log.info "exchange: ${exchange.dump()}"
            log.info "message: ${message.dump()}"
            MimeMultipart mimeMultipart = message.getBody()
            log.info "mimeMultipart: ${mimeMultipart.dump()}"
            Map<String, DataHandler> attachments = exchange.getIn().getAttachments()
            if (attachments.size() > 0) {
              for (String name : attachments.keySet()) {
                DataHandler dh = attachments.get(name);
                String filename = dh.getName();
                log.info "Attachement file name: ${filename}"
                InputStream objectData = dh.getInputStream();
                String destinationPath = "${filename}"
                Path target = Paths.get(destinationPath)
                Files.copy(objectData, target, REPLACE_EXISTING)
                target.toFile()
              }
            }
          } 
          })
        .to("log:newmail");
      }
      })
  }

  def start(){
    context.start()
  }

  def stop(){
    context.stop()
  }
  
}
```

To build the project

```bash
gradle build
```

To run the project

```bash
 java -jar mailbox-reader-0.0.1-SNAPSHOT.jar
```

To download the project

```bash
git clone https://github.com/josdem/camel-workshop.git
cd mailbox-reader
```

[Return to the main article](/techtalk/camel)


