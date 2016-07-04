+++
date = "2015-11-21T16:09:45-06:00"
draft = true
title = "Spring JavaMail and Freemarker"

+++

We're going to explore all flowing from controller to the freemarker html template to send an email.

```groovy
package com.jos.dem.jmailer.controller

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.stereotype.Controller
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.ResponseBody
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.http.ResponseEntity
import org.springframework.http.HttpStatus
import com.google.gson.Gson

import static org.springframework.web.bind.annotation.RequestMethod.GET
import static org.springframework.web.bind.annotation.RequestMethod.POST

import com.jos.dem.jmailer.service.EmailerService
import com.jos.dem.jmailer.command.MessageCommand
import com.jos.dem.jmailer.command.MessageType

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Controller
class EmailerController {

  @Autowired
  EmailerService emailerService

  Log log = LogFactory.getLog(this.class)

  @ResponseBody
  ResponseEntity<String> message(@RequestBody String json) {
    MessageCommand command = new Gson().fromJson(json, MessageCommand.class)
    log.info "Sending contact email: ${command.email}"
    command.type = MessageType.MESSAGE
    emailerService.sendEmail(command)
    new ResponseEntity<String>("OK", HttpStatus.OK)
  }

}
```

This controller receives an json as parameter and using Gson library parser transform it to a command, we could send a request from command line like this:


```bash
curl -H "Content-Type: application/json" -X POST -d '{"email":"joseluis.delacruz@gmail.com","message":"message"}' 'http://localhost:8080/jmailer/message'
```

The emailService is responsible to send an email based in a command, let's see its definition

```groovy
package com.jos.dem.jmailer.service

import com.jos.dem.jmailer.command.MessageCommand

interface EmailerService {

  def sendEmail(MessageCommand command)

}
```

First a interface definition to decouple controller and service relationship, then a definition.

```groovy
package com.jos.dem.jmailer.service.impl

import org.springframework.stereotype.Service
import org.springframework.beans.factory.annotation.Autowired

import com.jos.dem.jmailer.service.MessageService
import com.jos.dem.jmailer.service.EmailerService
import com.jos.dem.jmailer.command.MessageCommand
import com.jos.dem.jmailer.exception.EmailerException

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Service
class EmailerServiceImpl implements EmailerService {

  @Autowired
  MessageService messageService

  Log log = LogFactory.getLog(this.class)

  def sendEmail(MessageCommand command){
    log.info 'Sending email ${command.email}'
    messageService.message(command)
  }

}
```

Email service delegates to the messageService to send the email

```groovy
package com.jos.dem.jmailer.service

import com.jos.dem.jmailer.command.MessageCommand

interface MessageService {

  void message(final MessageCommand command)

}
```

Message service implementation

```groovy
package com.jos.dem.jmailer.service.impl

import javax.jms.JMSException
import javax.jms.Message
import javax.jms.ObjectMessage
import javax.jms.Session
import javax.jms.Destination

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.jms.core.JmsTemplate
import org.springframework.jms.core.MessageCreator
import org.springframework.stereotype.Service

import com.jos.dem.jmailer.service.MessageService
import com.jos.dem.jmailer.command.MessageCommand

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Service
class MessageServiceImpl implements MessageService {

  @Autowired
  JmsTemplate template
  @Autowired
  Destination destination

  Log log = LogFactory.getLog(getClass())

  void message(final MessageCommand command) {
    log.info "CALLING Message"

    template.send(destination, new MessageCreator() {
      Message createMessage(Session session) throws JMSException {
      ObjectMessage message = session.createObjectMessage()
      message.setObject(command)
      return message
      }
    })
  }
}
```
In our message service implementation we create an message and using an JMS template we deliver our message to the destination.

The receiver should be an MessageListener implementation

```groovy
package com.jos.dem.jmailer.messengine

import javax.jms.Message
import javax.jms.MessageListener
import javax.jms.ObjectMessage

import org.springframework.stereotype.Service
import org.springframework.beans.factory.annotation.Autowired

import com.jos.dem.jmailer.service.NotificationService

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Service
class JmsMessageListener implements MessageListener {

  @Autowired
  NotificationService notificationService

  Log log = LogFactory.getLog(getClass())

  void onMessage(Message message) {
    log.info 'Email message received'

    Object command =  ((ObjectMessage) message).getObject()
    notificationService.sendNotification(command)
  }

}
```

JmsMessageListener receives the email message and from this side uses and notificationService to transform it to an email

```groovy
package com.jos.dem.jmailer.service

import com.jos.dem.jmailer.command.MessageCommand

interface NotificationService {

  void sendNotification(MessageCommand command)

}
```

NotificationService implementation

```groovy
package com.jos.dem.jmailer.service.impl

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.beans.factory.annotation.Qualifier
import org.springframework.stereotype.Service

import com.jos.dem.jmailer.integration.MailService
import com.jos.dem.jmailer.service.NotificationService
import com.jos.dem.jmailer.command.MessageCommand
import com.jos.dem.jmailer.constant.ApplicationConstants

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Service
class NotificationServiceImpl implements NotificationService {

  @Autowired
  MailService mailService
  @Autowired
  Properties properties

  private Log log = LogFactory.getLog(getClass())

  @Override
  void sendNotification(MessageCommand messageCommand) {
    def (subject, templateName) = obtainSubjectAndResourceToSendNotification(messageCommand)
    def data = [email:messageCommand.email, subject:subject, templateName:templateName]
    mailService.sendMailWithTemplate(data, messageCommand.properties, data.templateName)
  }

  def obtainSubjectAndResourceToSendNotification(MessageCommand messageCommand){
    String templateKey = "${messageCommand.type.toString()}_PATH"
    String subjectKey = "${messageCommand.type.toString()}_SUBJECT"

    String templateName = properties.getProperty(templateKey)
    String subject = properties.getProperty(subjectKey)

    log.info("Sending email with subject: " + subject)
    [subject, templateName]
  }

}
```

The responsibility from this pice of code is to extract subject an a freemarker template name from a properties file and then send those parameters to the mail service

```groovy
package com.jos.dem.jmailer.integration

interface MailService {

  void sendMailWithTemplate(Map values, Map model, String template)

}
```

Mail service implementation

```groovy
package com.jos.dem.jmailer.integration.impl

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.mail.javamail.JavaMailSender
import org.springframework.mail.javamail.MimeMessageHelper
import org.springframework.mail.javamail.MimeMessagePreparator
import org.springframework.stereotype.Service
import org.springframework.ui.freemarker.FreeMarkerTemplateUtils

import com.jos.dem.jmailer.integration.MailService

import freemarker.template.Configuration
import freemarker.template.Template
import javax.mail.internet.MimeMessage

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Service
class MailServiceImpl implements MailService {

  @Autowired
  Configuration configuration

  @Autowired
  JavaMailSender javaMailSender

  Log log = LogFactory.getLog(getClass())

  void sendMailWithTemplate(final Map values, final Map model, final String template) {
    MimeMessagePreparator preparator = new MimeMessagePreparator() {
      void prepare(MimeMessage mimeMessage) throws Exception {
        MimeMessageHelper message = new MimeMessageHelper(mimeMessage, true)
        Template myTemplate = configuration.getTemplate(template)
        message.setTo(values.email)
        message.setSubject(values.subject)
        String text = FreeMarkerTemplateUtils.processTemplateIntoString(myTemplate, model)
        message.setText(text, true)
      }
    }

    this.javaMailSender.send(preparator)
  }

}
```

Mail service gets a template from a freemarker configuration, and prepare the email.

To get ths work, we need an xml context who defines javaMail parameters and freemarker configuration, as you can see we define in freemarker/mail path our templates

```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd">

  <context:component-scan base-package="com.jos.dem.jmailer.integration"/>
  <context:property-placeholder location="classpath:config/emailer.properties" />

  <bean id="javaMailSender" class="org.springframework.mail.javamail.JavaMailSenderImpl">
    <property name="host" value="smtp.gmail.com" />
    <property name="port" value="465" />
    <property name="username" value="${email.sender}" />
    <property name="password" value="${email.password}" />
    <property name="javaMailProperties">
      <props>
        <prop key="mail.smtp.auth">true</prop>
        <prop key="mail.smtp.socketFactory.port">465</prop>
        <prop key="mail.smtp.socketFactory.class">javax.net.ssl.SSLSocketFactory</prop>
        <prop key="mail.smtp.socketFactory.fallback">false</prop>
      </props>
    </property>
  </bean>

  <bean id="freemarkerConfiguration"
    class="org.springframework.ui.freemarker.FreeMarkerConfigurationFactoryBean">
    <property name="templateLoaderPath" value="classpath:/freemarker/mail/" />
  </bean>
</beans>
```

Our message template look like this

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Notificación</title>
  </head>

  <body>
    <h1>Mensaje</h1>
    <h3>Administrator</h3>
    <hr>
    <p>Message: ${message}</p>
  </body>

</html>
```

FreeMarker will step in and transform the template on-the-fly to plain HTML by replacing the ${message} with our command message information.

To download the project

```bash
git clone git@github.com:josdem/jmailer-bootstrap.git
git fetch
git checkout feature/java-mail
```

[Return to the main article](/techtalk/spring)
