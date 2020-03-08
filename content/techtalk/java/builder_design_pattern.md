+++
title =  "Builder Design Ppattern"
description = "Builder_design_pattern"
date = "2020-03-07T18:44:48-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

Immutable object is the one whose state cannot be modified after it is create, in functional programming is a fundamental piece, unfortunaltelly Java does not support functional programming by design so we need to implement it. Why is important immutability, well:

* Data will not change their state, so once it is verified it remainds valid.
* Thread safe
* No side effects

So since this object needs to be set/validated in creation, we can use builder design pattern which allow us to construct objects step by step and those could be with different attributes, let's consider the next user object:

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

As you can see, we can create an user object but we can not modify their values since we do not have setters on it. In builder pattern is common to have an inner static builder class, so that we can build an object and validate it.

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

    public static class Builder {

        private final String email;
        private final String password;
        private boolean active;
        private LocalDateTime lastLogin;

        public Builder(String email,
                           String password) {
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
