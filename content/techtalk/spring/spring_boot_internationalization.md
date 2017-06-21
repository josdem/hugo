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

As you can see we are creating an abstraction to use message source, so we only need to pass as parameter the message code. Here we are using it in a controller:

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
git fetch
git checkout feature/specific-locale
```

If you need to detect the locale from the client, then you need to create this implementation:

```groovy
package com.jos.dem.internationalization.helper

import org.springframework.stereotype.Component
import org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver

import javax.servlet.http.HttpServletRequest

import org.slf4j.Logger
import org.slf4j.LoggerFactory

@Component
class LocaleResolver extends AcceptHeaderLocaleResolver {

  private static final List<Locale> LOCALES = [new Locale("en"), new Locale("es")]

  Logger log = LoggerFactory.getLogger(this.class)

  @Override
  Locale resolveLocale(HttpServletRequest request) {
    if (!request.getHeader('Accept-Language')) {
      return Locale.getDefault()
    }
    List<Locale.LanguageRange> list = Locale.LanguageRange.parse(request.getHeader('Accept-Language'))
    Locale locale = Locale.lookup(list, LOCALES)
    return locale
  }

}
```

That's it, from the request we are reading the `Accept-Language` so we can know what is the preferred user's language for displaying pages.

So our `LocaleServiceImpl` now delegates the locale resolver to the previous class:

```groovy
package com.jos.dem.internationalization.services.impl

import org.springframework.context.MessageSource
import org.springframework.stereotype.Service
import org.springframework.beans.factory.annotation.Autowired

import javax.servlet.http.HttpServletRequest

import com.jos.dem.internationalization.services.LocaleService
import com.jos.dem.internationalization.helper.LocaleResolver

@Service
class LocaleServiceImpl implements LocaleService {

  @Autowired
  MessageSource messageSource
  @Autowired
  LocaleResolver localeResolver

  String getMessage(String code, HttpServletRequest request){
    messageSource.getMessage(code, null, localeResolver.resolveLocale(request))
  }

}
```

Don't forget to create `message_es.properties` to detect messages.

```properties
user.hello=¡Hola Internacionalización!
```

Finally this is our [Spock](http://spockframework.org/) testing case, to cover functionality:

```groovy
package com.jos.dem.internationalization

import javax.servlet.http.HttpServletRequest

import com.jos.dem.internationalization.helper.LocaleResolver

import spock.lang.Specification

class LocaleResolverSpec extends Specification {

  LocaleResolver resolver = new LocaleResolver()

  void "should get locale default"(){
    given:'A request'
      HttpServletRequest request = Mock(HttpServletRequest)
    when:'We call resolve'
      Locale result = resolver.resolveLocale(request)
    then:'We expect default'
      Locale.getDefault() == result
  }

  void "should get en-US as locale"(){
    given:'A request'
      HttpServletRequest request = Mock(HttpServletRequest)
    when:'We call resolve'
      request.getHeader('Accept-Language') >> 'en-US,en;q=0.8'
      Locale result = resolver.resolveLocale(request)
    then:'We expect default'
      result  == new Locale('en')
  }

  void "should get es-MX as locale"(){
    given:'A request'
      HttpServletRequest request = Mock(HttpServletRequest)
    when:'We call resolve'
      request.getHeader('Accept-Language') >> 'es-MX,en-US;q=0.7,en;q=0.3'
      Locale result = resolver.resolveLocale(request)
    then:'We expect default'
      result  == new Locale('es')
  }

}
```

To run the project:

```bash
gradle bootRun
```

To run the test:

```bash
gradle test
```

To download the project

```bash
git clone https://github.com/josdem/spring-boot-internationalization.git
git fetch
git checkout feature/detect-locale
```

[Return to the main article](/techtalk/spring)

