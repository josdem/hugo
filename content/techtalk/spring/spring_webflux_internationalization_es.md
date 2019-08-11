+++
title =  "Spring Webflux Internationalization"
description = "Spring webflux internationalization"
date = "2019-01-16T15:39:46-05:00"
tags = ["josdem", "techtalks","programming","technology","spring webflux"]
categories = ["techtalk", "code", "spring"]
+++

En este post técnico veremos como usar diferentes lenguajes (Inglés y Español) en una aplicación Spring Webflux junto con el template [Thymeleaf](https://www.thymeleaf.org/) o [REST](https://en.wikipedia.org/wiki/Representational_state_transfer). **NOTA:** Si quieres saber que necesitas tener instalado en tu computadora para crear fácilmente un proyecto de Spring boot, por favor visita mi previo post: [Spring Boot](/techtalk/spring_boot). Vamos a empezar creando un  nuevo proyecto de Spring Boot con Webflux como dependencia:

```bash
spring init --dependencies=webflux --build=gradle --language=java spring-webflux-internationalization
```

Aquí esta el `build.gradle` generado:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.7.RELEASE'
  id 'java'
}

apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.spring.webflux.internationalization'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
  mavenCentral()
}

dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-webflux'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
  testImplementation 'io.projectreactor:reactor-test'
}
```

Ahora agrega la dependencias Thymeleaf al archivo `build.gradle`

```groovy
implementation('org.thymeleaf:thymeleaf-spring5:3.0.11.RELEASE')
```

Spring Boot tiene una interfaz estratégica para resolver los mensajes para soporte de internacionalización de tales mensajes.

```java
@Bean
public MessageSource messageSource() {
  ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
  messageSource.setBasenames("i18n/messages");
  messageSource.setDefaultEncoding("UTF-8");
  return messageSource;
}
```

Ahora es tiempo de configurar el template resolver, template engine y el view resolver, desde que es lo que necesitamos para configurar Thymeleaf como un tempalte html.

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

Aquí está completa la configuración web.

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

Como puedes ver `SpringResourceTemplateResolver` está a cargo de definir los archivos y extensiones que usaremos. En este caso definiremos un archivo `index.html` como página web con un mensaje de saludo hello world.

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

Además `ResourceBundleMessageSource` define el path para cada lenguage soportado. En este case define `messages.properties` para el Inglés.

```properties
user.hello=Hello from internationalization!
```

Y `messages_es.properties` para el Español

```properties
user.hello=¡Hola Internacionalización!
```

El siguiente paseo es decirle a Spring como usar el Locale resolver. Así que nosotros necesitaremos agregar una clase que extienda de `DelegatingWebFluxConfiguration` y regrese un customizado `LocaleContextResolver`.

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

Aquí está nuestro locale resolver

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

Así es, estamos leyendo el lenguaje soportado en los headers de la respuesta al cliente, de esta manera podemos mostrar el mensaje adecuado si el ciente está usando Inglés o Español. Finalmente aquí está nuestro controller para poder desplegar la página web.

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

Para ejecutar el proyecto:

```bash
gradle bootRun
```

Después, ejecuta éste comando:

```bash
curl -v http://localhost:8080
```

Y deberías ver ésta salida:

```bash
* Rebuilt URL to: http://localhost:8080/
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> GET / HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Content-Type: text/html
< Content-Language: en-US
< content-length: 183
<
<html>
  <head>
    <meta charset="utf-8">
    <title>Internationalization with Spring Webflux</title>
  </head>
  <body>
  	<p>Hello from internationalization!</p>
  </body>
</html>
* Connection #0 to host localhost left intact
```

Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/spring-webflux-internationalization), para descargar el proyecto:

```bash
git clone git@github.com:josdem/spring-boot-internationalization.git
git fetch
git checkout thymeleaf
```

## Internacionalización en REST

Si necesitas implementar Internacionalización en REST en lugar de Thymeleaf, por favor considera los siguientes cambios.

```java
package com.jos.dem.spring.webflux.internationalization.handler;

import org.springframework.web.server.ServerWebExchange;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.spring.webflux.internationalization.service.LocaleService;

@RestController
public class InternationalizationController {

  @Autowired
  private LocaleService localeService;

  @GetMapping("/")
  public String index(ServerWebExchange exchange) {
    return localeService.getMessage("user.hello", exchange);
  }

}
```

Ahora estamos usando `@RestController` en lugar de `@Controller` y un servicio locale para resolver nuestros mensajes.

```java
package com.jos.dem.spring.webflux.internationalization.service;

import org.springframework.web.server.ServerWebExchange;

public interface LocaleService {
  String getMessage(String code, ServerWebExchange exchange);
}
```

Aquí está la implementación:

```java
package com.jos.dem.spring.webflux.internationalization.service.impl;

import org.springframework.stereotype.Service;
import org.springframework.context.MessageSource;
import org.springframework.context.i18n.LocaleContext;
import org.springframework.web.server.ServerWebExchange;
import org.springframework.beans.factory.annotation.Autowired;

import com.jos.dem.spring.webflux.internationalization.service.LocaleService;
import com.jos.dem.spring.webflux.internationalization.helper.LocaleResolver;

@Service
public class LocaleServiceImpl implements LocaleService {

  @Autowired
  private MessageSource messageSource;
  @Autowired
  private LocaleResolver localeResolver;

  @Override
  public String getMessage(String code, ServerWebExchange exchange) {
    LocaleContext localeContext = localeResolver.resolveLocaleContext(exchange);
    return messageSource.getMessage(code, null, localeContext.getLocale());
  }

}
```

Así es, nosotros estamos usando el mismo locale resolver de Thymeleaf pero ahora es un `@Component` así podemos usar `@Autowired` para inyectarlo.

```java
package com.jos.dem.spring.webflux.internationalization.helper;

import java.util.List;
import java.util.Locale;

import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import org.springframework.web.server.i18n.LocaleContextResolver;
import org.springframework.context.i18n.LocaleContext;
import org.springframework.context.i18n.SimpleLocaleContext;

@Component
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

Para ejecutar el proyecto:

```bash
gradle bootRun
```

Después, ejecuta éste comando:

```bash
curl -v http://localhost:8080
```

Y deberías ver ésta salida:

```bash
* Rebuilt URL to: http://localhost:8080/
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> GET / HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Content-Type: text/plain;charset=UTF-8
< Content-Length: 32
<
* Connection #0 to host localhost left intact
Hello from internationalization!%
```

Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/spring-webflux-internationalization), para descargar el proyecto:

```bash
git clone git@github.com:josdem/spring-boot-internationalization.git
git fetch
git checkout rest
```

[Return to the main article](/techtalk/spring#Spring_Boot_Reactive_es)
