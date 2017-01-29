+++
date = "2015-12-02T20:34:20-06:00"
draft = true
title = "Spring Boot and JMS"

+++

We’re going to explore all flowing from controller to the freemarker html template to send an email.

```groovy
package com.jos.dem.jmailer.controller

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.ResponseBody
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.stereotype.Controller
import org.springframework.http.ResponseEntity
import org.springframework.http.HttpStatus

import static org.springframework.web.bind.annotation.RequestMethod.GET
import static org.springframework.web.bind.annotation.RequestMethod.POST

import com.jos.dem.jmailer.service.EmailerService
import com.jos.dem.jmailer.command.MessageCommand

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Controller
class EmailerController {

  @Autowired
  EmailerService emailerService

  Log log = LogFactory.getLog(this.class)

  @RequestMapping(method = POST, value = "/message", consumes="application/json")
  @ResponseBody
  ResponseEntity<String> message(@RequestBody MessageCommand command) {
    log.info "Sending contact email: ${command.email}"
    emailerService.sendEmail(command)
    new ResponseEntity<String>("OK", HttpStatus.OK)
  }

}
```

This controller receives an json as parameter and using Gson library parser transform it to a command, we could send a request from command line like this:

```bash
curl -H "Content-Type: application/json" -X POST -d '{"email":"joseluis.delacruz@gmail.com","message":"message"}' 'http://localhost:8080/message'
```

The emailService is responsible to send an email based in a command, let’s see its definition

```groovy
package com.jos.dem.jmailer.service

import com.jos.dem.jmailer.command.MessageCommand

interface EmailerService {

  def sendEmail(MessageCommand command)

}
```

First a interface definition to decouple controller and service relationship, then an implementation.

```groovy
package com.jos.dem.jmailer.service.impl

import org.springframework.stereotype.Service
import org.springframework.beans.factory.annotation.Autowired

import com.jos.dem.jmailer.service.EmailerService
import com.jos.dem.jmailer.service.MessageService
import com.jos.dem.jmailer.command.MessageCommand

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

Message service implementation:

```groovy
package com.jos.dem.jmailer.service.impl

import javax.jms.JMSException
import javax.jms.ObjectMessage
import javax.jms.Message
import javax.jms.Session

import org.springframework.stereotype.Service
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.jms.annotation.EnableJms
import org.springframework.jms.core.JmsTemplate
import org.springframework.jms.core.MessageCreator

import com.jos.dem.jmailer.service.MessageService
import com.jos.dem.jmailer.command.MessageCommand

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Service
@EnableJms
class MessageServiceImpl implements MessageService {

  @Autowired
  JmsTemplate jmsTemplate

  Log log = LogFactory.getLog(this.class)

  void message(final MessageCommand command) {
    MessageCreator messageCreator = new MessageCreator() {

      @Override
      public Message createMessage(Session session) throws JMSException {
        ObjectMessage message = session.createObjectMessage()
        message.setObject(command)
        return message
      }
    }

    log.info 'Sending a new message'
    jmsTemplate.send("destination", messageCreator)
  }
}
```

In our message service implementation we create an message and using an JMS template we deliver our message to the destination.

The message receiver no need to implement any particular interface or for the method to have any particular name. Just the JmsListener annotation defines the name of the destination.

```groovy
package com.jos.dem.jmailer.messengine

import javax.jms.Message
import javax.jms.ObjectMessage

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.jms.annotation.JmsListener
import org.springframework.stereotype.Component
import org.springframework.util.FileSystemUtils

import com.jos.dem.jmailer.service.NotificationService

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Component
class JmsMessageListener {

  @Autowired
  NotificationService notificationService

  Log log = LogFactory.getLog(this.class)

  @JmsListener(destination = "destination", containerFactory = "myJmsContainerFactory")
  public void receiveMessage(Message message) {
    Object command =  ((ObjectMessage) message).getObject()
    log.info "Received ${command.dump()}"
    notificationService.sendNotification(command)
  }

}
```

JmsMessageListener receives the email message and from this side uses and notificationService to transform it to an email.

```groovy
package com.jos.dem.jmailer.service

import com.jos.dem.jmailer.command.MessageCommand

interface NotificationService {

  Boolean sendNotification(MessageCommand command)

}
```

NotificationService implementation:

```groovy
package com.jos.dem.jmailer.service.impl

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.beans.factory.annotation.Qualifier
import org.springframework.stereotype.Service

import com.jos.dem.jmailer.integration.MailService
import com.jos.dem.jmailer.service.NotificationService
import com.jos.dem.jmailer.command.MessageCommand
import org.springframework.beans.factory.annotation.Value

import org.apache.commons.logging.Log
import org.apache.commons.logging.LogFactory

@Service
class NotificationServiceImpl implements NotificationService {

  @Autowired
  MailService mailService

  @Value('${email.template}')
  String template
  @Value('${email.subject}')
  String subject

  private Log log = LogFactory.getLog(getClass())

  @Override
  Boolean sendNotification(MessageCommand messageCommand) {
     def data = [email:messageCommand.email, subject:subject]
     mailService.sendMailWithTemplate(data, messageCommand.properties, template)
  }

}
```

The responsibility from this pice of code is to extract subject an a freemarker template name from a properties file and then send those parameters to the mail service.

```groovy
package com.jos.dem.jmailer.integration

interface MailService {

  Boolean sendMailWithTemplate(Map values, Map model, String template)

}
```

Mail service implementation:

```groovy
package com.jos.dem.jmailer.service.impl

import org.springframework.beans.factory.annotation.Autowired
import org.springframework.mail.javamail.MimeMessageHelper
import org.springframework.mail.javamail.MimeMessagePreparator
import org.springframework.stereotype.Service
import org.springframework.ui.freemarker.FreeMarkerTemplateUtils
import org.springframework.mail.javamail.JavaMailSenderImpl

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
  JavaMailSenderImpl javaMailSender

  Log log = LogFactory.getLog(getClass())

  Boolean sendMailWithTemplate(final Map values, final Map model, final String template) {
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
    return true
  }

}
```

Mail service gets a template from a freemarker configuration, and prepare the email.

To get ths work, we need to define javaMail parameters and Jms container as Java configuration beans in our JmailerApplication.

```groovy
package com.jos.dem.jmailer

import javax.jms.ConnectionFactory

import org.springframework.boot.SpringApplication
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.context.ConfigurableApplicationContext
import org.springframework.context.annotation.Bean
import org.springframework.mail.javamail.JavaMailSenderImpl
import org.springframework.jms.config.JmsListenerContainerFactory
import org.springframework.jms.config.SimpleJmsListenerContainerFactory
import org.springframework.ui.freemarker.FreeMarkerConfigurationFactoryBean
import org.springframework.beans.factory.annotation.Value
import org.springframework.util.FileSystemUtils

@SpringBootApplication
class JmailerApplication {

  @Value('${email.username}')
  String username
  @Value('${email.password}')
  String password

  @Bean
  JmsListenerContainerFactory<?> myJmsContainerFactory(ConnectionFactory connectionFactory) {
    SimpleJmsListenerContainerFactory factory = new SimpleJmsListenerContainerFactory()
    factory.setConnectionFactory(connectionFactory)
    return factory
  }

  @Bean
  JavaMailSenderImpl javaMailSenderImpl(){
    JavaMailSenderImpl mailSender = new JavaMailSenderImpl()
    mailSender.setHost("smtp.gmail.com")
    mailSender.setPort(587)
    mailSender.setUsername(username)
    mailSender.setPassword(password)
    Properties prop = mailSender.getJavaMailProperties()
    prop.put("mail.transport.protocol", "smtp")
    prop.put("mail.smtp.auth", "true")
    prop.put("mail.smtp.starttls.enable", "true")
    prop.put("mail.debug", "true")
    return mailSender
  }

  static void main(String[] args) {
    SpringApplication.run(JmailerApplication.class, args)
  }

}
```

Our message template look like this:

File: $PROJECT_HOME/src/main/resources/templates/message.html

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
git clone https://github.com/josdem/jmailer-spring-boot.git
git fetch
git checkout feature/mail
```

[Return to the main article](/techtalk/spring)
