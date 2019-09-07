+++
title =  "Spring Boot Hazelcast"
description = "Hazelcast es una plataforma en memoria distribuida para manejo de datos y ejecuciones paralelas"
date = "2019-02-10T15:29:20-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

`java.util.Map` no es thread safe, si tú quieres usar un Mapa thread safe puedes usar `ConcurrentHashMap`, ¿Pero si quieres usar un mapa a través de multiples JVM?, entonces tu mejor opción es [Hazelcast](https://hazelcast.org/), además Hazelcast es Open Source escrita en Java y amigable con Maven. En este post técnico vamos a ver como podemos usar Hazelcast con Spring Boot. Si quieres saber más acerca de como crear una aplicación Spring Webflux por favor ve a mi previo post previo empezando con Spring Webflux [aquí](/techtalk/spring/spring_webflux_basics). Ahora, empecemos creando un nuevo proyecto con Spring Webflux como dependencia.

```bash
spring init --dependencies=webflux --build=gradle --language=java spring-boot-hazelcast
```

This is the `build.gradle` file generated:

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

group = 'com.jos.dem.springboot.hazelcast'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
  mavenCentral()
}

dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-webflux'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
  testImplementation 'io.projectreactor:reactor-test'
}
```

Next step is to add Hazelcast dependency:

```groovy
implementation 'com.hazelcast:hazelcast-spring'
```

Hazelcast can be configured through xml or using configuration api or even mix of both. Please consider this configuration using Java config.

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

To create a named `HazelcastInstance` you should set instance name of `Config` object. `MaxSizeConfig` is configuration for map's capacity. You can set a limit for number of entries or total memory cost of entries. `MaxSizePolicy` is maximum size policy in this case `FREE_HEAP_SIZE` is based on minimum free JVM heap memory in megabytes per JVM.

**Eviction**

Unless you delete the map entries manually or use an eviction policy, they will remain in the map. Hazelcast supports policy based eviction for distributed maps. Currently supported policies are LRU (Least Recently Used) and LFU (Least Frequently Used). Finally let's create a controller to store values in hazelcast and get them back.

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

That's it, if you start our Spring Boot application with this command:

```bash
gradle bootRun
```

You will be able to store a new value:

```bash
curl -X POST http://localhost:8080/hazelcast/write/key/value
```

And retrieve it

```bash
curl http://localhost:8080/hazelcast/read/key
```

Result:

```bash
2019-02-10 16:05 INFO --- [           main] o.s.b.web.embedded.netty.NettyWebServer : Netty started on port(s): 8080
2019-02-10 16:05 INFO --- [           main] c.j.d.s.hazelcast.HazelcastApplication  : Started HazelcastApplication in 6.08 seconds (JVM running for 6.459)
2019-02-10 16:05 INFO --- [ctor-http-nio-2] c.j.d.s.h.c.HazelcastController         : Storing key: key with value: value
2019-02-10 16:05 INFO --- [ctor-http-nio-2] c.h.i.p.impl.PartitionStateManager      : [192.168.0.8]:5701 [dev] [3.11.1] Initializing cluster partition table arrangement...
2019-02-10 16:05 INFO --- [ctor-http-nio-3] c.j.d.s.h.c.HazelcastController         : Reading stored value with key: key
```

To browse the project go [here](https://github.com/josdem/spring-boot-hazelcast), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-hazelcast.git
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
