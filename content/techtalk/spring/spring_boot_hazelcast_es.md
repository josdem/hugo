+++
title =  "Spring Boot Con Hazelcast"
description = "Hazelcast es una plataforma en memoria distribuida para manejo de datos y ejecuciones paralelas"
date = "2019-02-10T15:29:20-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

`java.util.Map` no es thread safe, si tú quieres usar un Mapa thread safe puedes usar `ConcurrentHashMap`, ¿Pero si quieres usar un mapa a través de multiples JVM?, entonces tu mejor opción es [Hazelcast](https://hazelcast.org/), además Hazelcast es Open Source escrita en Java y amigable con Maven. En este post técnico vamos a ver como podemos usar Hazelcast con Spring Boot. Si quieres saber más acerca de como crear una aplicación Spring Webflux por favor ve a mi previo post previo empezando con Spring Webflux [aquí](/techtalk/spring/spring_webflux_basics). Ahora, empecemos creando un nuevo proyecto con Spring Webflux como dependencia.

```bash
spring init --dependencies=webflux --build=gradle --language=java spring-boot-hazelcast
```

Aquí está el `build.gradle` generado:

```groovy
plugins {
  id 'org.springframework.boot' version '2.1.8.RELEASE'
  id 'io.spring.dependency-management' version '1.0.8.RELEASE'
  id 'java'
}

apply plugin: 'java'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management'

group = 'com.jos.dem.springboot.hazelcast'
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

El siguiente paso es agregar la dependencia de Hazelcast:

```groovy
implementation 'com.hazelcast:hazelcast-spring'
```

Hazelcast puede ser configurado através de XML o usando configuración Java incluso ambas. Por favor considera la siguiente configuración usando Java config.

```java
package com.jos.dem.springboot.hazelcast.conf;

import com.hazelcast.config.Config;
import com.hazelcast.config.EvictionPolicy;
import com.hazelcast.config.MapConfig;
import com.hazelcast.config.MaxSizeConfig;

import org.springframework.stereotype.Component;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Component
public class HazelcastConfiguration {

  @Bean
  public Config hazelCastConfig(){
    Config config = new Config();
    config.setInstanceName("hazelcast-instance")
      .addMapConfig(
          new MapConfig()
          .setName("configuration")
          .setMaxSizeConfig(new MaxSizeConfig(200, MaxSizeConfig.MaxSizePolicy.FREE_HEAP_SIZE))
          .setEvictionPolicy(EvictionPolicy.LRU)
          .setTimeToLiveSeconds(-1));
    return config;
  }

}
```

Para crear un`HazelcastInstance` deberías agregar un nombre al objeto `Config`. `MaxSizeConfig` es la configuración para la capacidad de nuestro mapa. Tú puedes configurar el máximo número de entradas así como el costo total de memoria. `MaxSizePolicy` es la política de máximo tamaño, en este caso `FREE_HEAP_SIZE` está basado en la cantidad mínima de memoria libre de la JVM como heap memory y se mide megabytes.

**Expulsión**

A menos que borres las entradas del mapa manualmente usando la política expulsión, estas permanecerán en el mapa. Hazelcast soporta la política de expulsión para mapas distribuidos. Actualmente las políticas soportadas son LRU (Los menos usados) and LFU (Los menos frecuentemente utilizados). Ahora, vamos a crear un controlador para almacenar nuestros valores en hazelcast y así obtenerlos de vuelta.

```java
package com.jos.dem.springboot.hazelcast.controller;

import java.util.Map;

import com.hazelcast.core.HazelcastInstance;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.beans.factory.annotation.Autowired;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@RestController
@RequestMapping("/hazelcast")
public class HazelcastController {

  @Autowired
  private HazelcastInstance hazelcastInstance;

  private Logger log = LoggerFactory.getLogger(this.getClass());

  @PostMapping("/write/{key}/{value}")
  public String write(@PathVariable("key") String key, @PathVariable("value") String value) {
    log.info("Storing key: {} with value: {}", key, value);
    Map<String, String> map = hazelcastInstance.getMap("memory");
    map.putIfAbsent(key, value);
    return "Key and value stored";
  }

  @GetMapping("/read/{key}")
  public String read(@PathVariable("key") String key){
    log.info("Reading stored value with key: {}", key);
    Map<String, String> map = hazelcastInstance.getMap("memory");
    return map.get(key);
  }

}
```

Así es, si inicias la aplicación Spring Boot con este comando:

```bash
gradle bootRun
```

Entonces podrás almacenar un nuevo valor así:

```bash
curl -X POST http://localhost:8080/hazelcast/write/key/value
```

Y obtenerlo así:

```bash
curl http://localhost:8080/hazelcast/read/key
```

Resultado:

```bash
2019-09-07 15:39:02.421  INFO 5934 --- [           main] com.hazelcast.core.LifecycleService      : [100.72.126.55]:5701 [dev] [3.11.4] [100.72.126.55]:5701 is STARTED
2019-09-07 15:39:03.029  INFO 5934 --- [           main] o.s.b.web.embedded.netty.NettyWebServer  : Netty started on port(s): 8080
2019-09-07 15:39:03.033  INFO 5934 --- [           main] c.j.d.s.hazelcast.HazelcastApplication   : Started HazelcastApplication in 5.518 seconds (JVM running for 5.867)
2019-09-07 15:45:32.573  INFO 5934 --- [or-http-epoll-2] c.j.d.s.h.c.HazelcastController          : Storing key: key with value: value
2019-09-07 15:45:32.593  INFO 5934 --- [or-http-epoll-2] c.h.i.p.impl.PartitionStateManager       : [100.72.126.55]:5701 [dev] [3.11.4] Initializing cluster partition table arrangement...
```

Para explorar el proyecto, por favor ve [aquí](https://github.com/josdem/spring-boot-hazelcast), para descargar el proyecto:

```bash
git clone git@github.com:josdem/spring-boot-hazelcast.git
```


[Regresar al artículo principal](/techtalk/spring#Spring_Boot_Reactive_es)
