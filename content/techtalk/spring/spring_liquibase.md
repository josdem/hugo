+++
title = "Spring Liquibase"
categories = ["techtalk", "code","spring framework"]
tags = ["josdem", "techtalks", "programming", "technology","spring framework"]
date = "2016-01-18T21:20:32-06:00"
description = "Liquibase can be run in a Spring environment by declaring a liquibase.spring.SpringLiquibase bean."
+++

Liquibase can be run in a Spring environment by declaring a liquibase.spring.SpringLiquibase bean.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:util="http://www.springframework.org/schema/util"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
  http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd
  http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

  <bean id="liquibase" class="liquibase.integration.spring.SpringLiquibase" lazy-init="false">
    <property name="dataSource" ref="dataSource" />
    <property name="changeLog" value="classpath:db-changelog.xml" />
  </bean>

</beans>
```

The root of all Liquibase changes is the databaseChangeLog file. In order to generate a initial changeLog file use this command:

```bash
liquibase --driver=com.mysql.jdbc.Driver --classpath=/path/mysql-connector-java-5.0.8-bin.jar --changeLogFile=/path/db-changelog.xml --url=jdbc:mysql://localhost:3306/db --username=yourUsername --password=yourPassword generateChangeLog
```

This will generate an db-changelog.xml file

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog" xmlns:ext="http://www.liquibase.org/xml/ns/dbchangelog-ext" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog-ext http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-ext.xsd http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.4.xsd">
    <changeSet author="josdem (generated)" id="1453158384521-1">
        <createTable tableName="BANK_CODE">
            <column name="bankCode" type="INT">
                <constraints nullable="false"/>
            </column>
            <column name="name" type="VARCHAR(255)"/>
        </createTable>
    </changeSet>
</databaseChangeLog>
```

If you want to include an sql file use this in your db-changelog.xml:

```xml
<include file="classpath:/migrations/load_data.sql"/>
```

File: load_data.sql

```sql
LOCK TABLES `BANK_CODE` WRITE;
INSERT INTO `BANK_CODE` VALUES ('2001','BANXICO');
UNLOCK TABLES;
```

This is an dataSource configuration in my application-context.xml notice the changelog-context.xml import

```xml
<import resource="classpath:/changelog-context.xml" />
<context:property-placeholder location="classpath:/config/persistence.properties" />

<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
  destroy-method="close">
  <property name="driverClassName" value="${jdbc.driverClassName}" />
  <property name="url" value="${jdbc.url}" />
  <property name="username" value="${jdbc.username}" />
  <property name="password" value="${jdbc.password}" />

  <!-- Adding pooled connections -->
  <property name="initialSize" value="${pooled.initialSize}" />
  <property name="maxActive" value="${pooled.maxActive}" />
  <property name="minIdle" value="${pooled.minIdle}" />
  <property name="maxIdle" value="${pooled.maxIdle}" />
  <property name="maxWait" value="${pooled.maxWait}" />
  <property name="timeBetweenEvictionRunsMillis" value="${pooled.timeBetweenEvictionRunsMillis}" />
  <property name="minEvictableIdleTimeMillis" value="${pooled.minEvictableIdleTimeMillis}" />
  <property name="validationQuery" value="${pooled.validationQuery}" />
  <property name="validationQueryTimeout" value="${pooled.validationQueryTimeout}" />
  <property name="testOnBorrow" value="${pooled.testOnBorrow}" />
  <property name="testWhileIdle" value="${pooled.testWhileIdle}" />
  <property name="testOnReturn" value="${pooled.testOnReturn}" />
</bean>
```

Don't forget to include your liquibase dependency in your build.gradle

```groovy
compile "org.liquibase:liquibase-core:3.4.2"
```

[Return to the main article](/techtalk/spring)



