+++
date = "2015-10-29T20:33:10-06:00"
title = "Grails 3 Externalized Configuration"
categories = ["techtalk","code","grails"]
tags = ["josdem","techtalks","programming","technology","grails"]
description = "Grails 3 externalization configuration strategy"
+++

In Grails 3 is not longer valid this configuration in the Config.groovy

```groovy
grails.config.locations = [ "file:${userHome}/.grails/${appName}-${Environment.current}-config.groovy" ]
```

The way we're externalizing our conguration is implementing setEnvironment in Application.groovy instead. There are many reasons why we're looking for externalize this:

* Every developer has its own configuration
* We can have several environments such as Development, QA, Stage and Production
* Its easy to modify any configuration
* Security
* Delegation of responsibilities

So, my implementations looks like this:

File: $GRAILS_APPLICATION_HOME/grails-app/init/application_name/Application.groovy

```groovy
package application_name

import grails.boot.GrailsApp
import grails.boot.config.GrailsAutoConfiguration

import org.springframework.beans.factory.config.YamlPropertiesFactoryBean
import org.springframework.context.EnvironmentAware
import org.springframework.core.env.Environment
import org.springframework.core.env.MapPropertySource
import org.springframework.core.env.PropertiesPropertySource
import org.springframework.core.io.FileSystemResource
import org.springframework.core.io.Resource

class Application extends GrailsAutoConfiguration implements EnvironmentAware {

  static void main(String[] args) {
    GrailsApp.run(Application, args)
  }

  @Override
  void setEnvironment(Environment environment) {
    def configBase = new File("${System.getProperty('user.home')}/.grails/application-${environment.activeProfiles[0]}.groovy")

    if(configBase.exists()) {
      println "Loading external configuration from Groovy: ${configBase.absolutePath}"
      def config = new ConfigSlurper().parse(configBase.toURL())
      environment.propertySources.addFirst(new MapPropertySource("externalGroovyConfig", config))
    } else {
      println "External config could not be found, checked ${configBase.absolutePath}"
    }
  }

}
```

setEnvironment will look for a file 'configBase' located in $USER_HOME/.grails/application-development.groovy, if exists will load its configuration. My groovy config file contains my database definition.

```groovy
dataSource {
  pooled = true
  dbCreate = "create-drop"
  driverClassName = "com.mysql.jdbc.Driver"
  username = "databaseUsername"
  password = "databasePassword"
  url = "jdbc:mysql://localhost/database"
  properties {
    jmxEnabled = true
    initialSize = 5
    maxActive = 50
    minIdle = 5
    maxIdle = 25
    maxWait = 10000
    maxAge = 10 * 60000
    timeBetweenEvictionRunsMillis = 5000
    minEvictableIdleTimeMillis = 60000
    validationQuery = "SELECT 1"
    validationQueryTimeout = 3
    validationInterval = 15000
    testOnBorrow = true
    testWhileIdle = true
    testOnReturn = false
    jdbcInterceptors = "ConnectionState"
    defaultTransactionIsolation = java.sql.Connection.TRANSACTION_READ_COMMITTED
  }
}
```

It is important to delete dataSource configuration from $GRAILS_APPLICATION_HOME/grails-app/conf/application.yml so Grails can use .groovy configuration file.

[Return to the main article](/techtalk/grails)


