+++
title =  "Builder Design Pattern"
description = "Builder_design_pattern"
date = "2020-03-07T18:44:48-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

An immutable object is one in which the state cannot be modified after it is created.  In functional programming, it is a fundamental piece, unfortunately, Java does not support functional programming by design so we need to implement it. Why is immutability important?, well:

* Data will not change their state, so once it is verified it remainds valid.
* Thread safe
* No side effects

So, since this object needs to be set/validated in creation, we can use the builder design pattern which allows us to construct objects step by step. These objects can even have different attributes. Let's consider the next user object:

```java
package com.jos.dem.builder;

import java.time.LocalDateTime;
import java.util.Optional;

public class User {
    private final String email;
    private final String password;
    private final boolean active;
    private final Optional<LocalDateTime> lastLogin;

    public User(String email, String password, boolean active, LocalDateTime lastLogin){
        this.email = email;
        this.password = password;
        this.active = active;
        this.lastLogin = Optional.ofNullable(lastLogin);
    }

    public String getEmail(){
        return this.email;
    }

    public String getPassword(){
        return this.password;
    }

    public boolean isActive(){
        return this.active;
    }

    public Optional<LocalDateTime> getLastLogin(){
        return this.lastLogin;
    }
}
```

As you can see, we can create a user object, but we cannot modify its values since we do not have setters on it. In builder pattern, it is common to have an inner static builder class so that we can build an object and validate it.

```java
package com.jos.dem.builder;

import java.time.LocalDateTime;
import java.util.Optional;

public class User {
    private final String email;
    private final String password;
    private final boolean active;
    private final Optional<LocalDateTime> lastLogin;

    private User(String email, String password, boolean active, LocalDateTime lastLogin){
        this.email = email;
        this.password = password;
        this.active = active;
        this.lastLogin = Optional.ofNullable(lastLogin);
    }

    public String getEmail(){
        return this.email;
    }

    public String getPassword(){
        return this.password;
    }

    public boolean isActive(){
        return this.active;
    }

    public Optional<LocalDateTime> getLastLogin(){
        return this.lastLogin;
    }

    public static class Builder {

        private final String email;
        private final String password;
        private boolean active;
        private LocalDateTime lastLogin;

        public Builder(String email, String password) {
            this.email = email;
            this.password = password;
        }

        public Builder active(boolean active) {
            this.active = active;
            return this;
        }

        public Builder lastLogin(LocalDateTime lastLogin) {
            this.lastLogin = lastLogin;
            return this;
        }

        public User build() {
            return new User(this.email,
                            this.password,
                            this.active,
                            this.lastLogin);
        }
    }
}
```

This is the test case for our user object

```java
package com.jos.dem.builder;

import static org.junit.jupiter.api.Assertions.assertTrue;

import java.time.LocalDateTime;

import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

class UserBuilderTest {

    @Test
    @DisplayName("should create basic user")
    void shouldCreateBasicUser() {
        User user = new User.Builder("josdem@email.com", "password")
            .build();

        assertEquals("josdem@email.com", user.getEmail(), "should have email");
        assertEquals("password", user.getPassword(), "should have password");
        assertFalse(user.isActive(), "should not be active");
    }

    @Test
    @DisplayName("should create an active user")
    void shouldCreateActiveUser() {
        User user = new User.Builder("josdem@email.com", "password")
            .active(true)
            .build();

        assertEquals("josdem@email.com", user.getEmail(), "should have email");
        assertEquals("password", user.getPassword(), "should have password");
        assertTrue(user.isActive(), "should be active");
    }

    @Test
    @DisplayName("should create an active user with last login date")
    void shouldCreateUserWithLastLoginTime() {
        User user = new User.Builder("josdem@email.com", "password")
            .active(true)
            .lastLogin(LocalDateTime.now())
            .build();

        assertEquals("josdem@email.com", user.getEmail(), "should have email");
        assertEquals("password", user.getPassword(), "should have password");
        assertTrue(user.isActive(), "should be active");
        assertTrue(LocalDateTime.now().compareTo(user.getLastLogin().get()) > 0, "should be active");
    }

}
```

[Lombok](https://projectlombok.org/) is a great tool to avoid boilerplate code, and it has a `@Builder` annotation.

```java
package com.jos.dem.builder;

import java.time.LocalDateTime;
import java.util.Optional;

import lombok.Getter;
import lombok.Builder;

@Getter
@Builder
public class UserLombok {
    private final String email;
    private final String password;
    private final boolean active;
    private final Optional<LocalDateTime> lastLogin;
}
```

That's it, see how Lombok is saving us a lot of code, so you can create a user object like this:


```java
UserLombok user = UserLombok
  .builder()
  .email("josdem@email.com")
  .password("password")
  .build();

```

But, what happens if I want to set email and passwords fields as required, just like our previous example? In that case, you can do it easily with Lombok `builderMethodName` annotation parameter.

```java
package com.jos.dem.builder;

import java.time.LocalDateTime;
import java.util.Optional;

import lombok.Getter;
import lombok.Builder;

@Getter
@Builder(builderMethodName = "requiredBuilder")
public class UserLombok {
    private final String email;
    private final String password;
    private final boolean active;
    private final Optional<LocalDateTime> lastLogin;

    public static UserLombokBuilder builder(String email, String password){
        return requiredBuilder()
                .email(email)
                .password(password);
    }
}
```

This is the test case:

```java
package com.jos.dem.builder;

import static org.junit.jupiter.api.Assertions.assertTrue;

import java.util.Optional;
import java.time.LocalDateTime;

import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;

class UserLombokBuilderTest {

    @Test
    @DisplayName("should create basic user")
    void shouldCreateBasicUser() {
        UserLombok user = UserLombok.builder("josdem@email.com", "password")
            .build();

        assertEquals("josdem@email.com", user.getEmail(), "should have email");
        assertEquals("password", user.getPassword(), "should have password");
        assertFalse(user.isActive(), "should not be active");
    }

    @Test
    @DisplayName("should create an active user")
    void shouldCreateActiveUser() {
        UserLombok user = UserLombok.builder("josdem@email.com", "password")
            .active(true)
            .build();

        assertEquals("josdem@email.com", user.getEmail(), "should have email");
        assertEquals("password", user.getPassword(), "should have password");
        assertTrue(user.isActive(), "should be active");
    }

    @Test
    @DisplayName("should create an active user with last login date")
    void shouldCreateUserWithLastLoginTime() {
        UserLombok user = UserLombok.builder("josdem@email.com", "password")
            .active(true)
            .lastLogin(Optional.of(LocalDateTime.now()))
            .build();

        assertEquals("josdem@email.com", user.getEmail(), "should have email");
        assertEquals("password", user.getPassword(), "should have password");
        assertTrue(user.isActive(), "should be active");
        assertTrue(LocalDateTime.now().compareTo(user.getLastLogin().get()) > 0, "should be active");
    }

}
```

To browse the code go [here](https://github.com/josdem/java-workshop), to download the code:

```bash
git clone https://github.com/josdem/java-workshop.git
cd builder
```

To run the code:

```bash
gradle test
```

[Return to the main article](/techtalk/java)

