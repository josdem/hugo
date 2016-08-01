+++
date = "2016-06-06T14:56:49-05:00"
draft = true
title = "Internationalization"

+++

Spring Boot uses the MessageSource configured with a MessageSourceAutoConfiguration. These settings can be easily changed in the `application.properties` file:

```
spring.messages.basename=i18n/messages
```

Once we create that line, you can define your application messages in the file `resources/i18n/messages.properties`:

```
user.hello=Hello from internationalization!
```

Spring boot create a MessageSource bean and is automatically added to the context, so you can use it in your services, let's create a service that use it.

```groovy
package com.jos.dem.internationalization.services

interface LocaleService {
  String getMessage(String code)
}
```

This is the implementation:

```groovy
package com.jos.dem.internationalization.services.impl

import org.springframework.context.MessageSource
import org.springframework.stereotype.Service
import org.springframework.beans.factory.annotation.Autowired

import com.jos.dem.internationalization.services.LocaleService

@Service
class LocaleServiceImpl implements LocaleService {

  @Autowired
  MessageSource messageSource

  String getMessage(String code){
    messageSource.getMessage(code, null, new Locale("en"))
  }

}
```

As you can see we are creating an abstration to use message source, so we only need to pass as parameter the message code. Here we are using it in a controller:

```groovy
package com.jos.dem.internationalization

import org.springframework.web.bind.annotation.RestController
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.beans.factory.annotation.Autowired

import com.jos.dem.internationalization.services.LocaleService

@RestController
class InternationalizationController {

  @Autowired
  LocaleService localeService

  @RequestMapping("/")
  String index(){
    localeService.getMessage('user.hello')
  }

}
```

We can run our project as follow:

```bash
gradle bootRun
```

To download the project

```bash
git clone https://github.com/josdem/spring-boot-internationalization.git
```

[Return to the main article](/techtalk/spring)

