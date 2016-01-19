+++
date = "2016-01-18T21:20:32-06:00"
draft = true
title = "Spring Liquibase"

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

[Return to the main article](/techtalk/spring)



