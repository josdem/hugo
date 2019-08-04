+++
title =  "JFairy"
description = "Jfairy is a Java fake data generator project "
date = "2019-08-04T15:32:05-04:00"
tags = ["josdem", "techtalks","programming","technology", "fairy", "java"]
categories = ["techtalk", "code", "java"]
+++

[Jfairy](https://devskiller.github.io/jfairy/) is a Java fake data generator project and this time I will show you how to use this library to generate random testing information data. First let's create an empty basic Java project using [lazybones](https://github.com/pledbrook/lazybones). In order to install lazybones in your computer, please use this tool: [sdkman](https://sdkman.io/). Then execute this command from your command line:

```bash
lazybones create java-basic jfairy-workshop
```

You should get this output:

```bash
Creating project from template java-basic (latest) in 'jfairy-workshop'

Java Application project template
------------------------------------

You have just created a basic Java application. There is a standard project
structure for source code and tests.

In this project you get:

* A Gradle build file
* A standard project structure:

    <proj>
      |
      +- src
          |
          +- main
          |     |
          |     +- java
          |
          +- test
          |   |
          |   +- java

Project created in jfairy-workshop!
```

Next step is to edit your `build.gradle` file and add JFairy, Junit5 and Apache Commons Lang3 dependencies

```groovy
implementation 'com.devskiller:jfairy:0.6.2'
implementation('org.apache.commons:commons-lang3:3.9')
testImplementation "org.junit.jupiter:junit-jupiter-api:$junitJupiterVersion"
testRuntime "org.junit.jupiter:junit-jupiter-engine:$junitJupiterVersion"
```

Now, please consider the following example how to create a fake data Company

```java
package com.jos.dem.jfairy.workshop;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assertions.assertNotNull;

import org.apache.commons.lang3.builder.ToStringBuilder;

import com.devskiller.jfairy.Fairy;
import com.devskiller.jfairy.producer.company.Company;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;

import java.util.logging.Logger;

class CompanyTest {

  private static final String VAT_REGEX = "[0-9]{2}-[0-9]{7}";
  private static final String DOMAIN_REGEX = "[a-zA-Z]+.[a-zA-Z]{2,10}";

  private Company company = Fairy.create().company();

  private static final Logger log = Logger.getLogger(CompanyTest.class.getName());

  @Test
  @DisplayName("Should validate company")
  void shouldValidateCompany() {
    log.info("Running: Should validate information from company: " + ToStringBuilder.reflectionToString(company));

    assertAll("company",
        () -> assertNotNull(company.getName(), "should get a name"),
        () -> assertTrue(company.getDomain().matches(DOMAIN_REGEX), "should have a valid domain"),
        () -> assertTrue(company.getVatIdentificationNumber().matches(VAT_REGEX), "should have a valid vat number")
        );
  }

}
```

Once you execute this test case, you should get an output similar to this one:

```bash
INFO: Running: Should validate information from company: com.devskiller.jfairy.producer.company.Company@612724ad[name=Adapt LLC,domain=adaptllc.biz,email=info,vatIdentificationNumber=83-0004026]
```

It is time to see another example, a person:

```java
package com.jos.dem.jfairy.workshop;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertTrue;
import static org.junit.jupiter.api.Assertions.assertNotNull;

import org.apache.commons.lang3.builder.ToStringBuilder;

import com.devskiller.jfairy.Fairy;
import com.devskiller.jfairy.producer.person.Person;
import com.devskiller.jfairy.producer.company.Company;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;

import java.util.logging.Logger;

class PersonTest {

  private static final String PHONE_REGEX = "[0-9]{3}-[0-9]{3}-[0-9]{3,4}";
  private static final String VAT_REGEX = "[0-9]{2}-[0-9]{7}";
  private static final String DOMAIN_REGEX = "[a-zA-Z]+.[a-zA-Z]{2,10}";

  private Person person = Fairy.create().person();

  private static final Logger log = Logger.getLogger(PersonTest.class.getName());

  @Test
  @DisplayName("Should validate person")
  void shouldValidatePerson() {
    log.info("Running: Should validate information from person: " + ToStringBuilder.reflectionToString(person));

    assertAll("person",
        () -> assertNotNull(person.getFirstName(), "Should get person's name"),
        () -> assertNotNull(person.getEmail(), "Should get person's email"),
        () -> assertTrue(person.getTelephoneNumber().matches(PHONE_REGEX), "Should have a valid telephone number")
        );
  }

  @Test
  @DisplayName("Should validate persons's company")
  void shouldValidatePersonCompany() {
    log.info("Running: Should validate information from person's company: " + ToStringBuilder.reflectionToString(person.getCompany()));

    Company company = person.getCompany();
    assertNotNull(company, () -> "should have a company");

    assertAll("company",
        () -> assertNotNull(company.getName(), "should get a name"),
        () -> assertTrue(company.getDomain().matches(DOMAIN_REGEX), "should have a valid domain"),
        () -> assertTrue(company.getVatIdentificationNumber().matches(VAT_REGEX), "should have a valid vat number")
        );
  }

}
```

This is the output:

```bash
INFO: Running: Should validate information from person: com.devskiller.jfairy.producer.person.Person@718121eb[address=175 Summer Place
San Francisco 58879,firstName=Brandon,middleName=,lastName=Le,email=le@gmail.com,username=brandonl,password=4boFv2ac,sex=MALE,telephoneNumber=488-324-672,dateOfBirth=1932-09-13,age=86,company=com.devskiller.jfairy.producer.company.Company@2f684683,companyEmail=brandon.le@buapel.biz,nationalIdentityCardNumber=404-92-0193,nationalIdentificationNumber=,passportNumber=lBplBWzhX,nationality=USA]
```

Certainly, you can build a person with specific criteria.

```java
package com.jos.dem.jfairy.workshop;

import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.apache.commons.lang3.builder.ToStringBuilder;

import com.devskiller.jfairy.Fairy;
import com.devskiller.jfairy.producer.person.Person;
import com.devskiller.jfairy.producer.person.PersonProperties;

import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.DisplayName;

import java.util.logging.Logger;

class PersonComposerTest {

  private static final Integer MIN_AGE=21;

  private Fairy fairy = Fairy.create();
  private Person adultMale = fairy.person(PersonProperties.male(), PersonProperties.minAge(MIN_AGE));

  private static final Logger log = Logger.getLogger(PersonComposerTest.class.getName());

  @Test
  @DisplayName("Should validate person")
  void shouldValidatePerson() {
    log.info("Running: Should validate information from person: " + ToStringBuilder.reflectionToString(adultMale));

    assertAll("adult",
        () -> assertTrue(adultMale.isMale(), "should be male"),
        () -> assertTrue(adultMale.getAge() >= MIN_AGE, "should be at least 21 years old")
        );
  }


}
```

To test the project:

```bash
gradle test
```

To browse the code go [here](https://github.com/josdem/jfairy-workshop), to download the code:

```bash
git clone https://github.com/josdem/jfairy-workshop.git
```

[Return to the main article](/techtalk/java)
