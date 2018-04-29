+++
title =  "Functional Interfaces"
tags = ["josdem","techtalks","programming","technology","java"]
categories = ["techtalk","code","java"]
date = "2018-04-28T20:22:39-05:00"
description = "A functional interface in Java is any with `@FunctionalInterface` annotation and with a SAM(Single Abstract Method) on it and it was introduced to facilitate Lambda functions."
+++

A functional interface in Java is any with `@FunctionalInterface` annotation and with SAM(Single Abstract Method). It was introduced to facilitate Lambda functions. Since a lambda function can only provide the implementation for one method, it is mandatory for the functional interfaces to have only one abstract method.

Java 8 has defined a lot of functional interfaces in `java.util.function` package. Some of them are `Consumer`, `Supplier`, `Function` and `Predicate`. 

`Consumer` has an `accept(Object)` method and represents an operation that accepts a single input argument and returns no result. Let's consider the following example:

```java
import java.util.function.Consumer;

public class ConsumerExample {

	public static void main(String[] args) {		
		Consumer<String> consumer = (x) -> System.out.println(x.toLowerCase());
		consumer.accept("JOSDEM");
	}

}
```

Output:

```bash
josdem
```

To download the code:

```bash
git clone https://github.com/josdem/java-workshop.git
cd functional_interfaces
```

To run the code:

```bash
javac ${JAVA_PROGRAM}.java
java ${JAVA_PROGRAM}
```

To test the code:

```bash
gradle test
```

[Return to the main article](/techtalk/java)