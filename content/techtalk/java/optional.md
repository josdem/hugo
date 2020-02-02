+++
title =  "Using Optional"
description = "Using Java Optional"
date = "2020-02-02T12:10:52-06:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

Once called a Billion-dollar mistake. `NullPointerException` is by far one of the most frequent exceptions in Java and since version 8 we can use it as a way to avoid this exception. [Optional](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html) is a container that can has a value or not. In this technical post we will review basic concepts about Optional. Please consider this example where we are creating an empty optional.

```java
@Test
void shouldCreateAnEmptyOptional() {
  Optional<String> empty = Optional.empty();
  assertFalse(empty.isPresent(), "should not be present");
}
```

Here is another way to create an optional

```java
@Test
@DisplayName("should validate if is present")
void shouldBePresentBy(){
  String nickname = "josdem"
  Optional<String> opt = Optional.of(nickname);
  assertTrue(opt.isPresent(), "should be present");
}
```

If some values could be null, you can use `ofNullable()` method.

```java
@Test
@DisplayName("should validate if is present by nullable")
  void shouldBePresentByNullable(){
  Optional<String> opt = Optional.ofNullable("josdem");
  assertTrue(opt.isPresent(), "should be present by nullable");
}
```

So, you can validate if a value is not null

```java
@Test
@DisplayName("should validate when is not present")
void shouldNotBePresent() {
  Optional<String> opt = Optional.ofNullable(null);
  assertFalse(opt.isPresent(), "should not be present");
}
```

You can take an action if value is present

```java
@Test
@DisplayName("should take action if is present")
void shouldPrintValue(){
  Optional<String> opt = Optional.ofNullable("josdem");
  opt.ifPresent(value -> log.info("value: " + opt.get()));
}
```

If a value is not present, you can return a default value

```java
@Test
@DisplayName("should display default value")
void shouldDisplayDefault(){
  String value = null;
  String name = Optional.ofNullable(value).orElse(StringUtils.EMPTY);
  assertEquals(StringUtils.EMPTY, name, "should be empty");
}
```

Or throwing an exception

```java
@Test
@DisplayName("should throw an exception")
  void shouldThrowAnException(){
  String value = null;
  assertThrows(RuntimeException.class, () -> Optional.ofNullable(value).orElseThrow(RuntimeException::new), "should throw an exception");
}
```

**Using Optional in Collections**

Here are some examples how to filter non-empty values in collections

```java
@Test
@DisplayName("should get a list non empty values")
void void shouldGetNonEmptyValuesList(){
  List<Optional<String>> staff = Arrays.asList(
    Optional.of("developer"), Optional.of("designer"), Optional.empty(), Optional.of("developer"));

  List<String> filteredStaff = staff.stream()
    .flatMap(Optional::stream)
    .collect(Collectors.toList());

  assertEquals(EXPECTED_VALUES, filteredStaff.size(), "should be 3 non empty values");
}
```

With `Optional::Stream` we are getting sequential stream of the only value present. Here is another way to do it

```java
@Test
@DisplayName("should filter by developer")
void shouldFilterByDeveloper(){
  List<Optional<String>> staff = Arrays.asList(
    Optional.of("developer"), Optional.of("designer"), Optional.empty(), Optional.of("developer"));

  List<String> developers = staff.stream()
    .filter(it -> it.isPresent())
    .map(it -> it.get())
    .filter(it -> it.equals("developer"))
    .collect(Collectors.toList());

  assertEquals(EXPECTED_DEVELOPERS, developers.size(), "should be two developers");
}
```

With `.filter` and `.map` we are doing the same as `Optional::Stream`, plus we are filtering only developer values.

To browse the code go [here](https://github.com/josdem/java-workshop), to download the code:

```bash
git clone https://github.com/josdem/java-workshop.git
cd optional
```

To run the code:

```bash
gradle test
```

[Return to the main article](/techtalk/java)
