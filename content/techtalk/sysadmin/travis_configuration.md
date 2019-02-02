+++
title =  "Travis CI Configuration"
description = "Travis CI is a hosted, distributed continuous integration service used to build and test software projects hosted at GitHub"
date = "2019-02-02T13:42:30-05:00"
tags = ["josdem", "techtalks","programming","technology","CI","Travis CI"]
categories = ["techtalk", "code","sysadmin","CI"]
+++

[Travis](https://travis-ci.com/) is a hosted, distributed continuous integration service used to build and test software projects hosted at [GitHub](https://github.com/). Travis is configured by adding a file named `.travis.yml` which is a [YAML](https://en.wikipedia.org/wiki/YAML) format text file, to the root directory of the repository. This is the most basic example:

```yml
language: java
jdk: openjdk8
```

Travis environment contains various versions of Open JDK, Oracle JDK, Gradle and Maven. Travis will search for a `pom.xml` or `build.gradle` file in order to use Maven or Gradle. Now for linking your GitHub account with Travis account, please follow this steps:

1. Go to [Travis-ci.com](https://travis-ci.com/) and sign up with GitHub.
2. Accept the Authorization of Travis. You’ll be redirected to GitHub.
3. Click the green Activate button, and select the repositories you want to use with Travis CI

In that way when you push a branch to your repository Travis will execute this steps:

1. Will clone your GitHub repository
2. `gradle assemble`
3. `gradle check`
4. `gradle test`

If previous tasks were successfully executed will generate a build success status:

[![Build Status](https://travis-ci.com/josdem/jugoterapia-webflux.svg?branch=master)](https://travis-ci.com/josdem/jugoterapia-webflux)

With Travis CI you can also do this tasks:

1. Run [JaCoCo](https://github.com/jacoco/jacoco) code coverage and fail the build if desired percentage is not met
2. Run [SonarQube](https://www.sonarqube.org/) code quality checks
3. Built [Docker](https://www.docker.com/) image and publish it


**Sonarqube Configuration**

In order to add Sonarqube you need to add this plugin to your `build.gradle` file:

```bash
plugins {
  id "org.sonarqube" version "2.7"
}
```

Next step is to set three environment variables in your Travis project configuration:

* SONAR_LOGIN
* SONAR_PROJECT
* SONAR_URL

Sonar login is an user token, in your Sonarqube account go to `User>My Account>Security` Your existing tokens are listed here, each with a Revoke button. Sonar project is your Project Key defined in the Sonar project main section and sonar url is your Sonar server.

<img src="/img/techtalks/sysadmin/travis.png">

Finally in your `.travis.yml` file you need to add the command required to start Sonarqube scanner process

```yml
language: java
jdk: openjdk8
script:
  - ./gradlew sonarqube
```

[![Quality Gate Status](https://sonar.josdem.io/api/project_badges/measure?project=com.jos.dem.jugoterapia.webflux%3Ajugoterapia-webflux&metric=alert_status)](https://sonar.josdem.io/dashboard?id=com.jos.dem.jugoterapia.webflux%3Ajugoterapia-webflux)

To browse a project using Travis configuration go [here](https://github.com/josdem/jugoterapia-webflux).

[Return to the main article](/techtalk/sysadmin)
