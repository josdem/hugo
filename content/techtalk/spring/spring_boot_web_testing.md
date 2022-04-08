+++
title =  "Spring Boot Testing the Web Layer"
description = "Spring Boot web layer testing using MockMvc"
date = "2022-04-08T15:23:34-04:00"
tags = ["josdem", "techtalks","programming","technology","Spring boot", "MockMvc"]
categories = ["techtalk", "code"]
+++

In this technical post we will go through the process of testing a web layer using [MockMvc](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/web/servlet/MockMvc.html). MockMvc helps to test Spring Boot controllers with auto configuration, if you want to know more about how to create Spring Boot application please go to my previous post getting started with Spring Boot [here](/techtalk/spring/spring_boot). As project target to test we will use [Spring Boot Swagger](/techtalk/spring/spring_swagger_boot_configuration) Please consider this controller.

```java
package com.josdem.swagger.controller;

import com.josdem.swagger.command.UserDto;
import com.josdem.swagger.model.User;
import com.josdem.swagger.service.UserService;
import java.util.List;
import javax.validation.ConstraintViolationException;
import javax.validation.Valid;
import javax.validation.constraints.Pattern;
import lombok.RequiredArgsConstructor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestController;

@Validated
@RestController
@RequestMapping("/users")
@RequiredArgsConstructor
public class UserController {

  private Logger log = LoggerFactory.getLogger(this.getClass());

  private final UserService userService;

  @GetMapping
  public List<User> getAll() {
    log.info("Getting all users");
    return userService.getAll();
  }

  @GetMapping(value = "/{uuid}")
  public User getByUuid(
      @PathVariable
          @Pattern(
              regexp =
                  "^[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{4}-[0-9a-fA-F]{12}$",
              message = "Invalid uuid format")
          String uuid) {
    log.info("Getting user by uuid: " + uuid);
    return userService.getByUuid(uuid);
  }

  @PostMapping(consumes = "application/json")
  public User create(@Valid @RequestBody UserDto userDto) {
    log.info("Saving user: w/uuid: " + userDto.getUuid());
    return userService.create(userDto);
  }

  @ResponseStatus(value = HttpStatus.BAD_REQUEST)
  @ExceptionHandler(ConstraintViolationException.class)
  public ResponseEntity<String> handleException(ConstraintViolationException ce) {
    return new ResponseEntity<>(ce.getMessage(), HttpStatus.BAD_REQUEST);
  }
}
```

As you can see we can register a new user with the post endpoint: `create`, so let's create a test case that creates a new user and hit this endpoint:

```java
@Test
@DisplayName("it creates a new user")
void shouldCreateUser(TestInfo testInfo) throws Exception {
  log.info("Running {}", testInfo.getDisplayName());
  User user = new User();
  user.setUuid(UUID.fromString("f35123c6-7dc0-4846-bb31-81f591079577"));
  user.setName("josdem");
  user.setEmail("contact@josdem.io");

  mockMvc.perform(post("/users")
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(new ObjectMapper().writeValueAsString(user)))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.uuid", matchesRegex(UUID_REGEX)))
            .andExpect(jsonPath("$.name").value("josdem"))
            .andExpect(jsonPath("$.email").value("contact@josdem.io"));
}
```

`MockMvc` is sending content as an object mapper using a Json Java library and handling the response using `MockMvcResultMatchers` so that, we can validate two important things: status code and body content. Here is the test case to validate we are able to get a user by `UUID`.

```java
@Test
@DisplayName("it gets user by uuid")
void shouldGetUserByUuid(TestInfo testInfo) throws Exception {
    log.info("Running: {}", testInfo.getDisplayName());
    mockMvc.perform(get("/users/f35123c6-7dc0-4846-bb31-81f591079577").contentType(MediaType.APPLICATION_JSON)).andExpect(status().isOk())
            .andExpect(jsonPath("$.uuid", matchesRegex(UUID_REGEX)))
            .andExpect(jsonPath("$.name").value("josdem"))
            .andExpect(jsonPath("$.email").value("contact@josdem.io"));
}
```

Since we have only one user registered in our application, we can validate user's collection size like:

```java
@Test
@DisplayName("it gets all users")
void shouldGetUsers(TestInfo testInfo) throws Exception {
    log.info("Running: {}", testInfo.getDisplayName());
    mockMvc.perform(get("/users").contentType(MediaType.APPLICATION_JSON)).andExpect(status().isOk())
            .andExpect(jsonPath("$.*", hasSize(1)));
}
```

That's it, to count users in our response we can use: `jsonPath("$.*", hasSize(1)`. Finally, let's validate a negative scenario where our endpoint is receiving an invalid `UUID`.

```java
@Test
@DisplayName("should return a bad request when we send an invalid UUID")
void shouldReturnBadResultWhenInvalidUuid(TestInfo testInfo) throws Exception {
    log.info("Running: {}", testInfo.getDisplayName());
    mockMvc.perform(get("/users/" + "invalidUuid").contentType(MediaType.APPLICATION_JSON)).andExpect(status().isBadRequest());
}
```

Here is the complete test case

```java
package com.josdem.swagger.controller;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.josdem.swagger.model.User;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.MethodOrderer;
import org.junit.jupiter.api.Order;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestInfo;
import org.junit.jupiter.api.TestMethodOrder;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import java.util.UUID;

import static org.hamcrest.Matchers.hasSize;
import static org.hamcrest.Matchers.matchesRegex;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@Slf4j
@SpringBootTest
@AutoConfigureMockMvc
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
public class UserControllerTest {

    private static final String UUID_REGEX = "[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}";
    private static final UUID USER_UUID = UUID.randomUUID();

    @Autowired
    private MockMvc mockMvc;

    @Test
    @Order(1)
    @DisplayName("it creates a new user")
    void shouldCreateUser(TestInfo testInfo) throws Exception {
        log.info("Running {}", testInfo.getDisplayName());
        User user = new User();
        user.setUuid(USER_UUID);
        user.setName("josdem");
        user.setEmail("contact@josdem.io");

        mockMvc.perform(post("/users")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(new ObjectMapper().writeValueAsString(user)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.uuid", matchesRegex(UUID_REGEX)))
                .andExpect(jsonPath("$.name").value("josdem"))
                .andExpect(jsonPath("$.email").value("contact@josdem.io"));
    }

    @Test
    @Order(2)
    @DisplayName("it gets user by uuid")
    void shouldGetUserByUuid(TestInfo testInfo) throws Exception {
        log.info("Running: {}", testInfo.getDisplayName());
        mockMvc.perform(get("/users/" + USER_UUID).contentType(MediaType.APPLICATION_JSON)).andExpect(status().isOk())
                .andExpect(jsonPath("$.uuid", matchesRegex(UUID_REGEX)))
                .andExpect(jsonPath("$.name").value("josdem"))
                .andExpect(jsonPath("$.email").value("contact@josdem.io"));
    }

    @Test
    @Order(3)
    @DisplayName("it gets all users")
    void shouldGetUsers(TestInfo testInfo) throws Exception {
        log.info("Running: {}", testInfo.getDisplayName());
        mockMvc.perform(get("/users").contentType(MediaType.APPLICATION_JSON)).andExpect(status().isOk())
                .andExpect(jsonPath("$.*", hasSize(1)));
    }

    @Test
    @Order(4)
    @DisplayName("should return a bad request when we send an invalid UUID")
    void shouldReturnBadResultWhenInvalidUuid(TestInfo testInfo) throws Exception {
        log.info("Running: {}", testInfo.getDisplayName());
        mockMvc.perform(get("/users/" + "invalidUuid").contentType(MediaType.APPLICATION_JSON)).andExpect(status().isBadRequest());
    }

}
```

In order to guarantee our test cases are executed in an particular order we are using this nice Junit5 annotation: `@Order`, if you want to know more how to use it please go to my previous technical post [Junit5](/techtalk/java/junit5/#Test_Execution_Order). To browse the code go [here](https://github.com/josdem/swagger-spring), to download the project:

```bash
git clone git@github.com:josdem/swagger-spring.git
cd boot-configuration
```

To run this project

```bash
gradle test
```

Using maven

```bash
mvn test
```

[Return to the main article](/techtalk/spring#Spring_Boot)
