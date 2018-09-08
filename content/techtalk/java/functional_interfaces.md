+++
title =  "Functional Interfaces"
tags = ["josdem","techtalks","programming","technology","java"]
categories = ["techtalk","code","java"]
date = "2018-04-28T20:22:39-05:00"
description = "A functional interface in Java is any with `@FunctionalInterface` annotation and with a SAM(Single Abstract Method) on it and it was introduced to facilitate Lambda functions."
+++

A functional interface in Java is any with `@FunctionalInterface` annotation and with SAM(Single Abstract Method). It was introduced to facilitate [Lambda expressions](/techtalk/java/lambda_expressions). Since a lambda function can only provide the implementation for one method, it is mandatory for the functional interfaces to have only one abstract method.

Java 8 has defined a lot of functional interfaces in `java.util.function` package. Some of them are `Consumer`, `Supplier`, `Function` and `Predicate`.

`Consumer` has an `accept(Object)` method and represents an operation that accepts a single input argument and returns no result. Let's consider the following example:

```java
import java.util.function.Consumer;

public class ConsumerExample {

  public static void main(String[] args) {
    Consumer<String> consumer = x -> System.out.println(x.toLowerCase());
    consumer.accept("JOSDEM");
  }

}
```

Output:

```bash
josdem
```

`Supplier` can be used in all contexts where there is no input but an output is expected. Functional method is `get()`. Let's consider the following example:

```java
package com.jos.dem.functional;

import static org.junit.jupiter.api.Assertions.assertEquals;

import org.junit.jupiter.api.Test;
import java.util.function.Supplier;

public class SupplierTest {

  @Test
  public void shouldSupplyAnString(){
    Supplier<String> supplier = () -> "josdem";
    assertEquals("josdem", supplier.get());
  }

}
```

`Function<T,R>` is used in all scenarios where an object T is the input, an operation is performed on it and and object R is returned as output. Functional method is `apply(T)`. Let's consider the following example:

```java
package com.jos.dem.functional;

import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.function.Function;
import org.junit.jupiter.api.Test;

public class FunctionTest {

  @Test
  public void shouldGetStringLenght(){
    Integer expectedResult = 6;
    Function<String, Integer> function = string -> string.length();
    assertEquals(expectedResult, function.apply("josdem"));
  }

}
```

 `andThen()` method combines the current Function instance with another one and returns a combined Function instance which applies the two functions in sequence.

 ```java
@Test
public void shouldKnowIfNicknameLengthIsEven(){
  Function<String, Integer> lenghtFunction = string -> string.length();
  Function<Integer, Boolean> evenFunction = integer -> integer % 2 == 0;

  assertTrue(lenghtFunction.andThen(evenFunction).apply("josdem"));
}
 ```

 Static method `Function.identity()` it just returns back the parameter which it gets as input.

 ```java
 @Test
public void shouldGetFunctionIdentity(){
  Function<String, String> function = Function.identity();
  assertEquals("josdem", function.apply("josdem"));
}
 ```

 That's it, in fact, we can write `x -> x` instead of `Function.identity()` and in some cases it’s clearer. Here is a list of common basic functions and how they can be written using lambda.

Runnable:

```java
() -> System.out.println("hello world");
```

Consumer:

```java
x -> System.out.println(x);
```

Supplier that returns a value:

```java
() -> "Hello World!";
```

Function that returns a constant k:

```java
x -> k;
```

`Predicate` is also a functional interface, therefore we can pass lambda expressions wherever predicate is expected. This is `filter()` formal definition: Returns a stream consisting of the elements of this stream that match the given predicate.

```java
Stream<T> filter(Predicate<? super T> predicate);
```

Let's consider the following example:

```java
private List<Person> persons = Arrays.asList(
	new Person("josdem", 5),
	new Person("tgrip", 4),
	new Person("edzero", 3),
	new Person("jeduan", 5),
	new Person("siedrix", 5)
);

@Test
public void shouldGetPersonsWithFourInRankingOrMore(){
	Predicate<Person> isHighRanked = person -> person.getRanking() >= 4;
	assertEquals(4, persons.stream().filter(isHighRanked).count());
}
```

`Person` is just a POJO with nickname as String and ranking as integer attributes. We can chain filters with predicate as the following example shows:

```java
@Test
public void shouldGetPersonsHihgRankedAndStartsWithJ(){
	Predicate<Person> isHighRanked = person -> person.getRanking() >= 4;
	Predicate<Person> startsWithJ = person -> person.getNickname().startsWith("j");
	assertEquals(2, persons.stream()
                    .filter(isHighRanked)
                    .filter(startsWithJ)
                    .count());
}
```

This is the complete test:

```java
package com.jos.dem.functional;

import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.List;
import java.util.Arrays;
import java.util.function.Predicate;
import org.junit.jupiter.api.Test;

public class PredicateTest {

	private List<Person> persons = Arrays.asList(
		new Person("josdem", 5),
		new Person("tgrip", 4),
		new Person("edzero", 3),
		new Person("jeduan", 5),
		new Person("siedrix", 5)
	);

	@Test
	public void shouldGetPersonsWithFourInRankingOrMore(){
		Predicate<Person> isHighRanked = person -> person.getRanking() >= 4;
		assertEquals(4, persons.stream().filter(isHighRanked).count());
	}

	@Test
	public void shouldGetPersonsHihgRankedAndStartsWithJ(){
		Predicate<Person> isHighRanked = person -> person.getRanking() >= 4;
		Predicate<Person> startsWithJ = person -> person.getNickname().startsWith("j");
		assertEquals(2, persons.stream()
                      .filter(isHighRanked)
                      .filter(startsWithJ)
                      .count());
	}

}
```


To download the code:

```bash
git clone https://github.com/josdem/java-workshop.git
cd functional-interfaces
```

To run the code:

```bash
javac ${JAVA_PROGRAM}.java
java ${JAVA_PROGRAM}
```

To test the code using Gradle:

```bash
gradle test
```

To test the code using Maven:

```bash
mvn test
```


[Return to the main article](/techtalk/java)
