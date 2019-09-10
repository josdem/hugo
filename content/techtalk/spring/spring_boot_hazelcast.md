+++
title =  "Spring Boot Hazelcast"
description = "Hazelcast is in-memory distributed computing platform for managing data and performing parallel execution for application speed and scale"
date = "2019-02-10T15:29:20-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

`java.util.Map` is not thread safe, if you want to use thread safe Map you can use `ConcurrentHashMap` what if you want to use a map through multiple JVM? then your best option is [Hazelcast](https://hazelcast.org/), besides Hazelcast is Open Source written in Java and Maven friendly. In this technical post we are going to view how we can use Hazelcast among with Spring Boot. If you want to know more about how to create Spring Webflux please go to my previous post getting started with Spring Webflux [here](/techtalk/spring/spring_webflux_basics). First let's create a new Spring Boot Webflux project:

```bash
spring init --dependencies=webflux --build=gradle --language=java spring-boot-hazelcast
```

This is the `build.gradle` file generated:

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

Next step is to add Hazelcast dependency:

```groovy
implementation 'com.hazelcast:hazelcast-spring'
```

Hazelcast can be configured through XML or using Java configuration or even mix of both. Please consider this configuration using Java config.

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

To create a named `HazelcastInstance` you should set an instance name in our `Config` object. `MaxSizeConfig` is configuration for map's capacity. You can set a limit for number of entries or total memory cost of entries. `MaxSizePolicy` is maximum size policy in this case `FREE_HEAP_SIZE` is based on minimum free JVM heap memory in megabytes.

**Eviction**

Unless you delete the map entries manually or use an eviction policy, they will remain in the map. Hazelcast supports policy based eviction for distributed maps. Currently supported policies are LRU (Least Recently Used) and LFU (Least Frequently Used). Now, let's create a controller to store values in hazelcast and get them back.

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
2019-09-07 15:39:02.421  INFO 5934 --- [           main] com.hazelcast.core.LifecycleService      : [100.72.126.55]:5701 [dev] [3.11.4] [100.72.126.55]:5701 is STARTED
2019-09-07 15:39:03.029  INFO 5934 --- [           main] o.s.b.web.embedded.netty.NettyWebServer  : Netty started on port(s): 8080
2019-09-07 15:39:03.033  INFO 5934 --- [           main] c.j.d.s.hazelcast.HazelcastApplication   : Started HazelcastApplication in 5.518 seconds (JVM running for 5.867)
2019-09-07 15:45:32.573  INFO 5934 --- [or-http-epoll-2] c.j.d.s.h.c.HazelcastController          : Storing key: key with value: value
2019-09-07 15:45:32.593  INFO 5934 --- [or-http-epoll-2] c.h.i.p.impl.PartitionStateManager       : [100.72.126.55]:5701 [dev] [3.11.4] Initializing cluster partition table arrangement...
```

To browse the project go [here](https://github.com/josdem/spring-boot-hazelcast), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-hazelcast.git
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
