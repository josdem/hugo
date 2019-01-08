+++
title = "Spring Boot Internationalization"
categories = ["techtalk", "code","spring boot"]
tags = ["josdem", "techtalks","programming","technology","spring boot"]
date = "2016-06-06T14:56:49-05:00"
description = "In this technical post, we will see how to manage different languages at your Spring Boot application aka. internationalization"
+++

In this technical post, we will see how to manage different languages at your Spring Boot application aka. internationalization. **NOTE:** If you need to know what tools you need to have installed in your computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot). Spring Boot uses `MessageSource` configured with a MessageSourceAutoConfiguration. These settings can be easily changed in the `application.properties` file:


```
spring.messages.basename=i18n/messages
```

Once we create that line, you can define your application messages in the file `resources/i18n/messages.properties`:

```
user.hello=Hello from internationalization!
```
Spring boot create a `MessageSource` bean and is automatically added to the context, so you can use it in your services. Let's create a new project using this command:

```bash
spring init --dependencies=web --build=gradle --language=java spring-boot-internationalization
```

Then we can create a `LocaleResolver` class that will be responsible for defining user’s locale.

```java
package com.jos.dem.springboot.internationalization.helper;

import java.util.List;
import java.util.Arrays;
import java.util.Locale;

import org.springframework.stereotype.Component;
import org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver;

import javax.servlet.http.HttpServletRequest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Component
public class LocaleResolver extends AcceptHeaderLocaleResolver {

  private static final List<Locale> LOCALES = Arrays.asList(new Locale("en"), new Locale("es"));

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @Override
  public Locale resolveLocale(HttpServletRequest request) {
    String language = request.getHeader("Accept-Language");
    if (language == null || language.isEmpty()) {
      return Locale.getDefault();
    }
    List<Locale.LanguageRange> list = Locale.LanguageRange.parse(language);
    Locale locale = Locale.lookup(list, LOCALES);
    return locale;
  }

}
```

That's it, here we have two locales supported: en and es. The locale should be passed in the header called "Accept-Language". In Spring Boot we can read that header using `HttpServletRequest`, please consider the following controller:


```java
package com.jos.dem.springboot.internationalization.controller;

import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.beans.factory.annotation.Autowired;

import javax.servlet.http.HttpServletRequest;
import com.jos.dem.springboot.internationalization.services.LocaleService;

@RestController
class InternationalizationController {

  @Autowired
  private LocaleService localeService;

  @RequestMapping("/")
  public String index(HttpServletRequest request){
    return localeService.getMessage("user.hello", request);
  }

}
```

As you can see we are creating an abstraction to use locale service and will be responsible for choosing right message according to specified locale.

```java
package com.jos.dem.springboot.internationalization.services;

import javax.servlet.http.HttpServletRequest;

public interface LocaleService {
  String getMessage(String code, HttpServletRequest request);
}
```

Here is the implementation:

```java
package com.jos.dem.springboot.internationalization.services.impl;

import org.springframework.stereotype.Service;
import org.springframework.context.MessageSource;
import org.springframework.beans.factory.annotation.Autowired;

import javax.servlet.http.HttpServletRequest;

import com.jos.dem.springboot.internationalization.helper.LocaleResolver;
import com.jos.dem.springboot.internationalization.services.LocaleService;

@Service
public class LocaleServiceImpl implements LocaleService {

  @Autowired
  private MessageSource messageSource;
  @Autowired
  private LocaleResolver localeResolver;

  public String getMessage(String code, HttpServletRequest request){
    return messageSource.getMessage(code, null, localeResolver.resolveLocale(request));
  }

}
```

Under the resources folder we created two files: `src/main/resources/i18n/messages.properties` and `src/main/resources/i18n/messages_es.properties`. Here is the content of `messages.properties`:

```properties
user.hello=Hello from internationalization!
```

And here is content of `messages_es.properties`:

```properties
user.hello=¡Hola Internacionalización!
```

Now you can run the project:

```bash
gradle bootRun
```

In your browser go to this url: [http://localhost:8080/](http://localhost:8080) and you should see the message based in your browser language.

Here is our Junit Jupiter testing case, to cover `LocaleResolver` functionality:

```java
package com.jos.dem.springboot.internationalization;

import static org.mockito.Mockito.when;
import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.Locale;
import javax.servlet.http.HttpServletRequest;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;

import org.mockito.Mock;
import org.mockito.InjectMocks;
import org.mockito.MockitoAnnotations;

import com.jos.dem.springboot.internationalization.helper.LocaleResolver;

class LocaleResolverTest {

  @InjectMocks
  private LocaleResolver resolver = new LocaleResolver();

  @Mock
  private HttpServletRequest request;

  @BeforeEach
  void setup() {
    MockitoAnnotations.initMocks(this);
  }

  @Test
  @DisplayName("should get locale default")
  void shouldGetDefaultLocale() {
    Locale result = resolver.resolveLocale(request);
    assertEquals(Locale.getDefault(), result);
  }

  @Test
  @DisplayName("should get en-US as locale")
  void shodldGetEnLocale() {
    when(request.getHeader("Accept-Language")).thenReturn("en-US,en;q=0.8");
    Locale result = resolver.resolveLocale(request);
    assertEquals(new Locale("en"), result);
  }

  @Test
  @DisplayName("should get es-MX as locale")
  void shouldGetEsLocale() {
    when(request.getHeader("Accept-Language")).thenReturn("es-MX,en-US;q=0.7,en;q=0.3");
    Locale result = resolver.resolveLocale(request);
    assertEquals(new Locale("es"), result);
  }

}
```

To browse the code go [here](https://github.com/josdem/spring-boot-internationalization), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-internationalization.git
```

To run the test:

```bash
gradle test
```

[Return to the main article](/techtalk/spring#Spring_Boot)
