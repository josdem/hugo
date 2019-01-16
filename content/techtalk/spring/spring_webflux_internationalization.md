+++
title =  "Spring Webflux Internationalization"
description = "Spring_webflux_internationalization"
date = "2019-01-16T15:39:46-05:00"
tags = ["josdem", "techtalks","programming","technology","spring webflux"]
categories = ["techtalk", "code", "spring"]
+++

In this technical post, we will see how to use different languages in your Spring Webflux application along with [Thymeleaf](https://www.thymeleaf.org/) template framework. **NOTE:** If you need to know what tools you need to have installed in your computer in order to create a Spring Boot basic project, please refer my previous post: [Spring Boot](/techtalk/spring_boot). Let's create a new project using this command:

```bash
spring init --dependencies=webflux --build=gradle --language=java spring-webflux-internationalization
```

This is the `build.gradle` generated file:

```groovy
buildscript {
  ext {
    springBootVersion = '2.1.2.RELEASE'
  }
  repositories {
    mavenCentral()
  }
  dependencies {
    classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
  }
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.spring.webflux.internationalization'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
	mavenCentral()
}

dependencies {
  implementation('org.springframework.boot:spring-boot-starter-webflux')
  testImplementation('org.springframework.boot:spring-boot-starter-test')
  testImplementation('io.projectreactor:reactor-test')
}
```

Then add Thymeleaf dependency to your `build.gradle` file

```groovy
implementation('org.thymeleaf:thymeleaf-spring5:3.0.11.RELEASE')
```

Spring Boot has an strategy interface for resolving messages, with support for internationalization of such messages.

```java
@Bean
public MessageSource messageSource() {
  ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
  messageSource.setBasenames("i18n/messages");
  messageSource.setDefaultEncoding("UTF-8");
  return messageSource;
}
```

Now is time to configure template resolver, template engine and a view resolver since is what we need in order to configure Thymeleaf as html template.

```java
@Bean
public ITemplateResolver thymeleafTemplateResolver() {
  final SpringResourceTemplateResolver resolver = new SpringResourceTemplateResolver();
  resolver.setApplicationContext(this.context);
  resolver.setPrefix("classpath:templates/");
  resolver.setSuffix(".html");
  resolver.setTemplateMode(TemplateMode.HTML);
  resolver.setCacheable(false);
  resolver.setCheckExistence(false);
  return resolver;
}

@Bean
public ISpringWebFluxTemplateEngine thymeleafTemplateEngine() {
  SpringWebFluxTemplateEngine templateEngine = new SpringWebFluxTemplateEngine();
  templateEngine.setTemplateResolver(thymeleafTemplateResolver());
  return templateEngine;
}

@Bean
public ThymeleafReactiveViewResolver thymeleafReactiveViewResolver() {
  ThymeleafReactiveViewResolver viewResolver = new ThymeleafReactiveViewResolver();
  viewResolver.setTemplateEngine(thymeleafTemplateEngine());
  return viewResolver;
}

@Override
public void configureViewResolvers(ViewResolverRegistry registry) {
  registry.viewResolver(thymeleafReactiveViewResolver());
}
```

Here is the complete complete web configuration

```java
package com.jos.dem.spring.webflux.internationalization.config;

import org.springframework.context.MessageSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.reactive.config.EnableWebFlux;
import org.springframework.web.reactive.config.WebFluxConfigurer;
import org.springframework.context.support.ResourceBundleMessageSource;
import org.springframework.web.reactive.config.ViewResolverRegistry;

import org.thymeleaf.templatemode.TemplateMode;
import org.thymeleaf.templateresolver.ITemplateResolver;
import org.thymeleaf.spring5.SpringWebFluxTemplateEngine;
import org.thymeleaf.spring5.ISpringWebFluxTemplateEngine;
import org.thymeleaf.spring5.view.reactive.ThymeleafReactiveViewResolver;
import org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver;

@Configuration
@EnableWebFlux
public class WebConfig implements ApplicationContextAware, WebFluxConfigurer {

  private ApplicationContext context;

  @Override
  public void setApplicationContext(ApplicationContext context) {
    this.context = context;
  }

  @Bean
  public MessageSource messageSource() {
    ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
    messageSource.setBasenames("i18n/messages");
    messageSource.setDefaultEncoding("UTF-8");
    return messageSource;
  }

  @Bean
  public ITemplateResolver thymeleafTemplateResolver() {
    final SpringResourceTemplateResolver resolver = new SpringResourceTemplateResolver();
    resolver.setApplicationContext(this.context);
    resolver.setPrefix("classpath:templates/");
    resolver.setSuffix(".html");
    resolver.setTemplateMode(TemplateMode.HTML);
    resolver.setCacheable(false);
    resolver.setCheckExistence(false);
    return resolver;
  }

  @Bean
  public ISpringWebFluxTemplateEngine thymeleafTemplateEngine() {
    SpringWebFluxTemplateEngine templateEngine = new SpringWebFluxTemplateEngine();
    templateEngine.setTemplateResolver(thymeleafTemplateResolver());
    return templateEngine;
  }

  @Bean
  public ThymeleafReactiveViewResolver thymeleafReactiveViewResolver() {
    ThymeleafReactiveViewResolver viewResolver = new ThymeleafReactiveViewResolver();
    viewResolver.setTemplateEngine(thymeleafTemplateEngine());
    return viewResolver;
  }

  @Override
  public void configureViewResolvers(ViewResolverRegistry registry) {
    registry.viewResolver(thymeleafReactiveViewResolver());
  }

}
```

As you can see `SpringResourceTemplateResolver` is in charge to set our template files path and extension. In this case will will define an `index.html` web page with a hello world message.

```html
<html>
  <head>
    <meta charset="utf-8">
    <title>Internationalization with Spring Webflux</title>
  </head>
  <body>
  	<p th:text="#{user.hello}"></p>
  </body>
</html>
```

Also `ResourceBundleMessageSource` is defining message resources path for each supported language. In this case we will define `messages.properties` for English.

```properties
user.hello=Hello from internationalization!
```

And `messages_es.properties` for Spanish

```properties
user.hello=¡Hola Internacionalización!
```

Next step is to tell Spring to use a Locale resolver. So we need to add a configuration class which extends from `DelegatingWebFluxConfiguration` and returns our custom `LocaleContextResolver`.

```java
package com.jos.dem.spring.webflux.internationalization.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.server.i18n.LocaleContextResolver;
import org.springframework.web.reactive.config.DelegatingWebFluxConfiguration;

import com.jos.dem.spring.webflux.internationalization.helper.LocaleResolver;

@Configuration
public class LocaleSupportConfig extends DelegatingWebFluxConfiguration {

  @Override
  protected LocaleContextResolver createLocaleContextResolver() {
    return new LocaleResolver();
  }

}
```

Here is our locale resolver

```java
package com.jos.dem.spring.webflux.internationalization.helper;

import java.util.List;
import java.util.Locale;

import org.springframework.web.server.ServerWebExchange;
import org.springframework.web.server.i18n.LocaleContextResolver;
import org.springframework.context.i18n.LocaleContext;
import org.springframework.context.i18n.SimpleLocaleContext;

public class LocaleResolver implements LocaleContextResolver {

  @Override
  public LocaleContext resolveLocaleContext(ServerWebExchange exchange) {
    String language = exchange.getRequest().getHeaders().getFirst("Accept-Language");

    Locale targetLocale = Locale.getDefault();
    if (language != null && !language.isEmpty()) {
      targetLocale = Locale.forLanguageTag(language);
    }
    return new SimpleLocaleContext(targetLocale);
  }

  @Override
  public void setLocaleContext(ServerWebExchange exchange, LocaleContext localeContext) {
    throw new UnsupportedOperationException("Not Supported");
  }

}
```

That's it, we are reading language support in the headers from client request, in that way we can show the right message to our client whether is using English or Spanish. Finally here is our controller to render the index web page.

```java
package com.jos.dem.spring.webflux.internationalization.handler;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class InternationalizationController {

  @GetMapping("/")
  public String index() {
    return "index";
  }

}
```

To run the project:

```bash
gradle bootRun
```

Then go to this address: [http://localhost:8080](http://localhost:8080). To browse the project go [here](https://github.com/josdem/spring-webflux-internationalization), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-internationalization.git
```

[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
