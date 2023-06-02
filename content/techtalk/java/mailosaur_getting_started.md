+++
title =  "Mailosaur Getting Started"
description = "Mailosaur_getting_started"
date = "2023-06-02T09:22:39-04:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++


If you want to create end-to-end tests that rely on email verification such as sent, confirmation, password reset, etc. [Mailosaur](https://mailosaur.com/) is your friend. **Note:** You will need a Mailosaur account in order to create a new trial account; sign up [here](https://mailosaur.com/app/signup/). In this technical post, we will go over the required configuration to set up an email sent validation. Let's start by creating a new Gradle - Java project with Kotlin DSL script :) `build.gradle.kts` should look like this:

```kotlin
plugins {
    java
}

val mailosaurVersion: String by project
val junitJupiterVersion: String by project

group = "com.josdem.mailosaur"
version = "1.0-SNAPSHOT"

repositories {
    mavenCentral()
}

dependencies {
    implementation("com.mailosaur:mailosaur-java:$mailosaurVersion")
    testImplementation("org.junit.jupiter:junit-jupiter-api:$junitJupiterVersion")
    testRuntimeOnly("org.junit.jupiter:junit-jupiter-engine")
}

tasks.getByName<Test>("test") {
    useJUnitPlatform()
    systemProperty("mailosaurApiKey", System.getProperty("mailosaurApiKey"))
    systemProperty("mailosaurServerId", System.getProperty("mailosaurServerId"))
}
```

This is a cool approach using `gradle.properties` to define version values and refer those in the DSL script. Also we are reading the [API key](https://mailosaur.com/docs/managing-your-account/api-keys/) and server ID required to authenticate to Mailosaur, you can see all this data at the [Mailosaur Dashboard](https://mailosaur.com/app/servers/). Now create a Junit5 test case to validate an email in your inbox.

```java
package com.josdem.mailosaur;

import com.mailosaur.MailosaurClient;
import com.mailosaur.MailosaurException;
import com.mailosaur.models.Message;
import com.mailosaur.models.MessageSearchParams;
import com.mailosaur.models.SearchCriteria;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

import java.io.IOException;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertNotNull;

class MailosaurSenderTest {

    private static final String SERVER_DOMAIN = ".mailosaur.net";

    private String apiKey = "YOUR_API_KEY";
    private String serverId = "SERVER_ID";

    @BeforeEach
    void setup() {
        serverId = System.getProperty("mailosaurServerId");
        apiKey = System.getProperty("mailosaurApiKey");
    }

    @Test
    @DisplayName("Sending email")
    void shouldSendEmail() throws MailosaurException, IOException {
        MailosaurClient mailosaur = new MailosaurClient(apiKey);
        MessageSearchParams params = new MessageSearchParams();
        params.withServer(serverId);

        SearchCriteria criteria = new SearchCriteria();
        criteria.withSentTo("josdem@" + serverId + SERVER_DOMAIN);
        Message message = mailosaur.messages().get(params, criteria);

        assertNotNull(message);
        assertEquals("test", message.subject());
    }
}
```

To browse the code go [here](https://github.com/josdem/mailosaur-workshop), to download the code:

```bash
git clone git@github.com:josdem/mailosaur-workshop.git
```

[Return to the main article](/techtalk/java)