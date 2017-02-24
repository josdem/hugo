+++
date = "2017-02-24T16:01:07-06:00"
title = "Spring Boot Logback"
tags = ["josdem","techtalks","programming","technology","logback"]
categories = ["techtalk","code"]

+++

Logback is a logging framework wheter console or file, is nowadays the best option for logging your Spring Boot application. [Here](https://logback.qos.ch/) you can find more information about it. This time I will show you how to configure Logback.

First you need to add logback into gradle as dependencies.

```groovy
compile "ch.qos.logback:logback-classic:1.2.1"
compile "ch.qos.logback:logback-core:1.2.1"
```

Then under `${PROJECT_HOME}/src/main/resources` create this `logback.groovy` file:

```groovy
import ch.qos.logback.core.ConsoleAppender
import ch.qos.logback.core.rolling.RollingFileAppender
import ch.qos.logback.classic.encoder.PatternLayoutEncoder

import static ch.qos.logback.classic.Level.DEBUG
import static ch.qos.logback.classic.Level.INFO

appender("STDOUT", ConsoleAppender) {
  encoder(PatternLayoutEncoder) {
    pattern = "%d{HH:mm:ss.SSS} [%thread] %-5level %logger{5} Groovy - %msg%n"
  }
}

logger("com.jos.dem.vetlog", INFO)
logger('org.springframework', WARN)
root(WARN, ["STDOUT"])
```

 Logback will give preference to groovy configuration over xml configuration.

 If you want to send your loggin to a file, use this Groovy structure:

 ```groovy
 import ch.qos.logback.core.ConsoleAppender
import ch.qos.logback.core.rolling.RollingFileAppender
import ch.qos.logback.classic.encoder.PatternLayoutEncoder

import static ch.qos.logback.classic.Level.DEBUG
import static ch.qos.logback.classic.Level.INFO

appender("ROLLING", RollingFileAppender) {
  encoder(PatternLayoutEncoder) {
    pattern = "%d{HH:mm:ss.SSS} [%thread] %-5level %logger{5} Groovy - %msg%n"
  }
  rollingPolicy(TimeBasedRollingPolicy){
    FileNamePattern = "vetlog-spring-boot-%d{yyyy-MM}.log"
  }
}

logger("com.jos.dem.vetlog", INFO)
logger('org.springframework', WARN)
root(WARN, ["ROLLING"])
 ```

 That's it, when you run your project logback will send logging to console or file.

 To download the project:

```bash
git clone https://github.com/josdem/vetlog-spring-boot.git
git fetch
git checkout feature/27
```

To run the project, please read the [wiki](https://github.com/josdem/vetlog-spring-boot/wiki/YAML%20File) and execute this command:

```bash
gradle bootRun -Dspring.config.location=$HOME/.vetlog-spring-boot/application-development.yml
```

[Return to the main article](/techtalk/spring)
