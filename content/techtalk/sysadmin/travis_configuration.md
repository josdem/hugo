+++
title =  "Travis Configuration"
description = "Travis CI is a hosted, distributed continuous integration service used to build and test software projects hosted at GitHub"
date = "2019-02-02T13:42:30-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
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

With important build information like this one: [Build Status Passing](https://travis-ci.com/josdem/jugoterapia-webflux)

With Travis CI you can also do this tasks:

1. Run [JaCoCo](https://github.com/jacoco/jacoco) code coverage and fail the build if desired percentage is not met
2. Run [SonarQube](https://www.sonarqube.org/) code quality checks
3. Built [Docker](https://www.docker.com/) image and publish it


To browse a project using Travis configuration go [here](https://github.com/josdem/jugoterapia-webflux).

[Return to the main article](/techtalk/sysadmin)