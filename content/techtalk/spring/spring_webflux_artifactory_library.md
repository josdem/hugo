+++
title =  "Spring Webflux Publishing an Artifactory Library"
description = "How to pulish a Jar library in Jfrog artifactory"
date = "2022-05-12T15:47:21-04:00"
tags = ["josdem", "techtalks","programming","technology","JFrog","Artifactory"]
categories = ["techtalk", "code"]
+++

In this technical post, we will go over the process of publish a Spring Webflux library to [JFrog](https://jfrog.com/). **NOTE:** If you want to know more about creating a Spring Webflux application, please go to my previous post getting started with Spring Webflux [here](/techtalk/spring/spring_webflux_basics). Let's begin creating a new Spring Boot project with Webflux and Lombok.

```bash
spring init --dependencies=webflux,lombok --build=gradle --language=java juice-webflux
```

Now, let's activate Maven publish plugin:

```bash
apply plugin: 'maven-publish'
```

And add our Gradle publishing task definition.

```bash
publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
            versionMapping {
                usage('java-api') {
                    fromResolutionOf('runtimeClasspath')
                }
                usage('java-runtime') {
                    fromResolutionResult()
                }
            }
        }
    }
    repositories {
        maven {
            url = "https://jfrog.josdem.io/artifactory/libs-snapshot-local"
            allowInsecureProtocol = true
            credentials {
                username = "${artifactory_user}"
                password = "${artifactory_password}"
            }
        }
    }
}
```
In this publishing section, we are defining, artifact id and group id taken from our `build.gradle` definition; versioning is handled by `versionMapping` DSL, which allows configuring version strategies; if you want to know more about it, please go [here](https://docs.gradle.org/7.4.2/userguide/publishing_maven.html#publishing_maven:resolved_dependencies). JFrog Platform hosts local, virtual and remote [repository types](https://www.jfrog.com/confluence/display/JFROG/Repository+Management); in this case, we are using `libs-snapshot-local` to publish our library. This is our complete `build.gradle` file.

```groovy
plugins {
    id 'org.springframework.boot' version '2.6.7'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
}

ext {
    artifactory_user = "$System.env.ARTIFACTORY_USER"
    artifactory_password = "$System.env.ARTIFACTORY_PASSWORD"
}

apply plugin: 'maven-publish'

group = 'com.josdem.jugoterapia.webclient'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '16'

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
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'io.projectreactor:reactor-test'
    testCompileOnly 'org.projectlombok:lombok'
    testAnnotationProcessor 'org.projectlombok:lombok'
}

publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
            versionMapping {
                usage('java-api') {
                    fromResolutionOf('runtimeClasspath')
                }
                usage('java-runtime') {
                    fromResolutionResult()
                }
            }
        }
    }
    repositories {
        url = "https://jfrog.josdem.io/artifactory/libs-snapshot-local"
        maven {
            credentials {
                username = "${artifactory_user}"
                password = "${artifactory_password}"
            }
        }
    }
}

tasks.named('test') {
    useJUnitPlatform()
}
```

If you want to see a complete example how to publish with extra information, please go [here](https://docs.gradle.org/7.4.2/userguide/publishing_maven.html#publishing_maven:complete_example). We are passing artifactory credentials using system properties, so publishing from the command line will be

```bash
export ARTIFACTORY_USER=${username}
export ARTIFACTORY_PASSWORD=${password}
gradle publish
```

where:

- `${username}` Is artifactory username
- `${password}` Is artifactory password

## Publishing using Maven

With Maven you need to specify in the `distributionManagement` section repository id, which in this case should be `snapshots` and URL. Plus repositories, specifying your artifactory server:

```xml
<distributionManagement>
    <repository>
        <id>snapshots</id>
        <url>https://jfrog.josdem.io/artifactory/libs-snapshot-local</url>
    </repository>
</distributionManagement>
<repositories>
    <repository>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
        <id>central</id>
        <name>libs-release</name>
        <url>https://jfrog.josdem.io/artifactory/libs-release</url>
    </repository>
    <repository>
        <snapshots/>
        <id>snapshots</id>
        <name>libs-snapshot</name>
        <url>https://jfrog.josdem.io/artifactory/libs-snapshot</url>
    </repository>
</repositories>
```

Plus, your Jfrog credentials in the `${USER_HOME}/.m2/settings.xml` file

```xml
<settings>
  <servers>
    <server>
      <id>artifactory</id>
      <username>username</username>
      <password>password</password>
    </server>
  </servers>
</settings>
```

Here is the complete `pom.xml` file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.7</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.josdem.jugoterapia.webclient</groupId>
    <artifactId>juice-webclient</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>Jugoterapia client library</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>16</java.version>
    </properties>
    <distributionManagement>
        <repository>
            <id>snapshots</id>
            <url>https://jfrog.josdem.io/artifactory/libs-snapshot-local</url>
        </repository>
    </distributionManagement>
    <repositories>
        <repository>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
            <id>central</id>
            <name>libs-release</name>
            <url>https://jfrog.josdem.io/artifactory/libs-release</url>
        </repository>
        <repository>
            <snapshots/>
            <id>snapshots</id>
            <name>libs-snapshot</name>
            <url>https://jfrog.josdem.io/artifactory/libs-snapshot</url>
        </repository>
    </repositories>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-webflux</artifactId>
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
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

To publish library to artifactory

```bash
mvn deploy
```

If you want to learn more and publish your own library, feel free to drop me a message on my home page website and ask for a Jfrog credentials. To browse the project go [here](https://github.com/josdem/juice-webclient), to download the project:

```bash
git clone git@github.com:josdem/juice-webclient.git
```

[Return to the main article](/techtalk/spring#Spring_Boot_Reactive)
