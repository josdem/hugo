+++
title = "Spring Boot Liquibase"
categories = ["techtalk","code","spring boot"]
tags = ["josdem","techtalks","programming","technology","spring boot"]
date = "2017-02-17T12:07:56-06:00"
description = "Liquibase is a source control for your databases. This time I will show you how to add Liquibase to a Spring Boot project. First you need to add your Liquibase dependency in your build.gradle file"
+++

Liquibase is a source control for your databases. This time I will show you how to add Liquibase to a Spring Boot project. First you need to add your Liquibase dependency in your `build.gradle` file

```groovy
compile "org.liquibase:liquibase-core:3.5.3"
```

**Concepts**

*Changelog:* Developers store database changes in text-based files on their local development machines and apply them to their local databases.

*ChangeSet:* Are uniquely identified by the "author" and "id" attribute. Each changeset generally contains a change which describes the change/refactoring to apply to the database.

Then execute this command in order to generate the master changelog.

```bash
liquibase --driver=com.mysql.jdbc.Driver --classpath=/${CONNECTOR_PATH}/mysql-connector-java-5.1.34.jar --changeLogFile=${PROJECT_PATH}/src/main/resources/db/changelog/db.changelog-master.yaml --url=jdbc:mysql://localhost:3306/db_name --username=username --password=password generateChangeLog
```

If you are using Windowns computer:

```bash
liquibase --driver=com.mysql.jdbc.Driver --classpath=C:\${CONNECTOR_PATH}\mysql-connector-java-5.1.34.jar --changeLogFile=C:\${PROJECT_PATH}\db\changelog\db.changelog-master.yaml --url=jdbc:mysql://localhost:3306/db_name --username=username--password=password generateChangeLog
```

Previous command will generate a `db.changelog-master.yaml` file.

```yaml
databaseChangeLog:
- changeSet:
    id: 1487128064865-1
    author: josdem (generated)
    changes:
    - createTable:
        columns:
        - column:
            autoIncrement: true
            constraints:
              primaryKey: true
            name: id
            type: BIGINT
        - column:
            constraints:
              nullable: false
            name: date_created
            type: datetime(6)
        - column:
            constraints:
              nullable: false
            name: email
            type: VARCHAR(255)
        - column:
            constraints:
              nullable: false
            name: status
            type: INT
        - column:
            constraints:
              nullable: false
            name: token
            type: VARCHAR(255)
        tableName: registration_code
- changeSet:
    id: 1487128064865-2
    author: josdem (generated)
    changes:
    - createTable:
        columns:
        - column:
            autoIncrement: true
            constraints:
              primaryKey: true
            name: id
            type: BIGINT
        - column:
            constraints:
              nullable: false
            name: account_non_expired
            type: BIT(1)
        - column:
            constraints:
              nullable: false
            name: account_non_locked
            type: BIT(1)
        - column:
            constraints:
              nullable: false
            name: credentials_non_expired
            type: BIT(1)
        - column:
            constraints:
              nullable: false
            name: date_created
            type: datetime(6)
        - column:
            name: email
            type: VARCHAR(255)
        - column:
            constraints:
              nullable: false
            name: enabled
            type: BIT(1)
        - column:
            name: firstname
            type: VARCHAR(255)
        - column:
            name: lastname
            type: VARCHAR(255)
        - column:
            constraints:
              nullable: false
            name: password
            type: VARCHAR(255)
        - column:
            constraints:
              nullable: false
            name: role
            type: VARCHAR(255)
        - column:
            constraints:
              nullable: false
            name: username
            type: VARCHAR(255)
        tableName: user
- changeSet:
    id: 1487128064865-3
    author: josdem (generated)
    changes:
    - addUniqueConstraint:
        columnNames: username
        constraintName: UK_sb8bbouer5wak8vyiiy4pf2bx
        tableName: user

```

This are my current entities which Liquibase took in order to create the YAML file:

**User.groovy**

```groovy
package com.jos.dem.vetlog.model

import static javax.persistence.EnumType.STRING
import static javax.persistence.GenerationType.AUTO

import java.io.Serializable
import javax.persistence.Id
import javax.persistence.Column
import javax.persistence.Entity
import javax.persistence.Enumerated
import javax.persistence.GeneratedValue

@Entity
class User implements Serializable {

  @Id
  @GeneratedValue(strategy=AUTO)
  Long id
  @Column(unique = true, nullable = false)
  String username
  @Column(nullable = false)
  String password
  @Column(nullable = true)
  String firstname
  @Column(nullable = true)
  String lastname
  @Column(nullable = true)
  String email
  @Column(nullable = false)
  @Enumerated(STRING)
  Role role

  @Column(nullable = false)
  Boolean enabled = false
  @Column(nullable = false)
  Boolean accountNonExpired = true
  @Column(nullable = false)
  Boolean credentialsNonExpired = true
  @Column(nullable = false)
  Boolean accountNonLocked = true
  @Column(nullable = false)
  Date dateCreated = new Date()

}
```

**RegistrationCode.groovy**

```groovy
package com.jos.dem.vetlog.model

import static javax.persistence.GenerationType.AUTO

import javax.persistence.Id
import javax.persistence.Entity
import javax.persistence.Column
import javax.persistence.GeneratedValue

import com.jos.dem.vetlog.enums.RegistrationCodeStatus

@Entity
class RegistrationCode {

  @Id
  @GeneratedValue(strategy=AUTO)
  Long id
  @Column(nullable = false)
  String email
  @Column(nullable = false)
  Date dateCreated = new Date()
  @Column(nullable = false)
  String token = UUID.randomUUID().toString().replaceAll('-','')
  @Column(nullable = false)
  RegistrationCodeStatus status = RegistrationCodeStatus.VALID

  Boolean isValid(){
    status == RegistrationCodeStatus.VALID ? true : false
  }
}
```

**Role.groovy**

```groovy
package com.jos.dem.vetlog.model

enum Role {
  USER,ADMIN
}
```

That's it, when you start your application Liquibase will execute the master changelog and will create the database schema for you.


To download the project:

```bash
git clone https://github.com/josdem/vetlog-spring-boot.git
git fetch
git checkout feature/23
```

To run the project, please read the [wiki](https://github.com/josdem/vetlog-spring-boot/wiki/YAML%20File) and execute this command:

```bash
gradle bootRun -Dspring.config.location=$HOME/.vetlog-spring-boot/application-development.yml
```


[Return to the main article](/techtalk/spring)
