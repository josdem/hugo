+++
title =  "Spring Boot Hazelcast"
description = "Hazelcast is in-memory distributed computing platform for managing data and performing parallel execution for application speed and scale"
date = "2019-02-10T15:29:20-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

`java.util.Map` is not thread safe, if you want to use thread safe Map you can use `ConcurrentHashMap` what if you want to use a map through multiple JVM? then your best option is [Hazelcast](https://hazelcast.org/), besides Hazelcast is Open Source written in Java and Maven friendly. In this technical post we are going to view how we can use Hazelcast among with Spring Boot. If you want to know more about how to create Spring Webflux please go to my previous post getting started with Spring Webflux [here](/techtalk/spring/spring_webflux_basics). First let's create a new Spring Boot Webflux project:

```bash
spring init --dependencies=webflux,lombok --build=gradle --language=java spring-boot-hazelcast
```

This is the `build.gradle` file generated:

```groovy
plugins {
  id 'org.springframework.boot' version '2.3.4.RELEASE'
  id 'io.spring.dependency-management' version '1.0.10.RELEASE'
  id 'java'
}

group = 'com.jos.dem.springboot.hazelcast'
version = '1.0.0-SNAPSHOT'
sourceCompatibility = '13'

configurations {
  compileOnly {
    extendsFrom annotationProcessor
  }
}

repositories {
  mavenCentral()
}

dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-webflux'
  compileOnly 'org.projectlombok:lombok'
  annotationProcessor 'org.projectlombok:lombok'
  testImplementation('org.springframework.boot:spring-boot-starter-test') {
    exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
  }
  testImplementation 'io.projectreactor:reactor-test'
}

test {
  useJUnitPlatform()
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
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

@Component
public class HazelcastConfiguration {

    @Bean
    public Config hazelCastConfig() {
        return new Config().setInstanceName("hazelcast-instance")
                .addMapConfig(
                        new MapConfig()
                                .setName("configuration")
                                .setMaxSizeConfig(new MaxSizeConfig(200, MaxSizeConfig.MaxSizePolicy.FREE_HEAP_SIZE))
                                .setEvictionPolicy(EvictionPolicy.LRU)
                                .setTimeToLiveSeconds(-1));
    }

}
```

To create a named `HazelcastInstance` you should set an instance name in our `Config` object. `MaxSizeConfig` is configuration for map's capacity. You can set a limit for number of entries or total memory cost of entries. `MaxSizePolicy` is maximum size policy in this case `FREE_HEAP_SIZE` is based on minimum free JVM heap memory in megabytes.

**Eviction**

Unless you delete the map entries manually or use an eviction policy, they will remain in the map. Hazelcast supports policy based eviction for distributed maps. Currently supported policies are LRU (Least Recently Used) and LFU (Least Frequently Used). Now, let's create a controller to store values in hazelcast and get them back.

```java
package com.jos.dem.springboot.hazelcast.controller;

import com.hazelcast.core.HazelcastInstance;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Map;

@Slf4j
@RestController
@RequestMapping("/hazelcast")
@RequiredArgsConstructor
public class HazelcastController {

  private final HazelcastInstance hazelcastInstance;

  @PostMapping("/write/{key}/{value}")
  public String write(@PathVariable("key") String key, @PathVariable("value") String value) {
    log.info("Storing key: {} with value: {}", key, value);
    Map<String, String> map = hazelcastInstance.getMap("memory");
    map.putIfAbsent(key, value);
    return "Key and value stored";
  }

  @GetMapping("/read/{key}")
  public String read(@PathVariable("key") String key) {
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
2020-09-17 21:19:52.864  INFO 18154 --- [           main] com.hazelcast.core.LifecycleService      : [10.0.0.253]:5701 [dev] [3.12.9] [10.0.0.253]:5701 is STARTED
2020-09-17 21:19:53.142  INFO 18154 --- [           main] o.s.b.web.embedded.netty.NettyWebServer  : Netty started on port(s): 8080
2020-09-17 21:19:53.148  INFO 18154 --- [           main] c.j.d.s.hazelcast.HazelcastApplication   : Started HazelcastApplication in 3.721 seconds (JVM running for 3.911)
2020-09-17 21:19:57.296  INFO 18154 --- [or-http-epoll-2] c.j.d.s.h.c.HazelcastController          : Storing key: key with value: value
2020-09-17 21:19:57.308  INFO 18154 --- [or-http-epoll-2] c.h.i.p.impl.PartitionStateManager       : [10.0.0.253]:5701 [dev] [3.12.9] Initializing cluster partition table arrangement...
2020-09-17 21:20:06.127  INFO 18154 --- [or-http-epoll-3] c.j.d.s.h.c.HazelcastController          : Reading stored value with key: key
```

**Using Maven**

You can do the same using Maven, the only difference is that you need to specify `--build=maven` parameter in the `spring init` command line:

```bash
spring init --dependencies=webflux,lombok --language=java --build=maven spring-boot-hazelcast
```

This is the pom.xml file generated:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.jos.dem.springboot.hazelcast</groupId>
    <artifactId>spring-boot-hazelcast</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <name>Spring Boot Hazelcast</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>13</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
        </dependency>

        <dependency>
            <groupId>com.hazelcast</groupId>
            <artifactId>hazelcast-spring</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

To run the project with Maven:

```bash
mvn spring-boot:run
```


To browse the project go [here](https://github.com/josdem/spring-boot-hazelcast), to download the project:

```bash
git clone git@github.com:josdem/spring-boot-hazelcast.git
```


[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
