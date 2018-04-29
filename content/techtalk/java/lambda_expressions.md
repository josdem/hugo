+++
tags = ["josdem","techtalks","programming","technology","java","lambda"]
categories = ["techtalk","code","java"]
date = "2016-12-17T10:05:56-06:00"
title = "Lambda Expressions"
description = "Lambda expression is an argument list follow by an arrow token and a body or code expression."
+++

Lambda expression is an argument list follow by an arrow token and a body or code expression.

Example 1:

```java
(x, y)  ->  x + y;
```

Example 2:

```java
() -> "Hello World!";   
```

Example 3:

```java
x -> x.toLowerCase();
```

*Lambdas should be an expression, not a narrative*

**Avoid Specifying Parameter Types:**

Type to the parameters is optional and can be omitted.

Do this:

```java
(x, y) -> x + y;
```

Instead of this:

```java
(Integer x, Integer y) -> x + y;
```

**Avoid Parentheses Around a Single Parameter:**

Do this:

```java
x -> x.toLowerCase();
```

Instead of this:

```java
(x) -> x.toLowerCase();
```

**Avoid Return Statement and Braces:**

Braces and return statements are optional in one-line lambda bodies.

Do this:

```java
x -> x.toLowerCase();
```

Instead of this:

```java
x -> { return x.toLowerCase() };
```

**Use Method References:**

It is very useful to use another Java 8 feature: [method references](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html).

So, the lambda expression:

```java
string -> string.toLowerCase();
```

Could be substituted by:

```java
String::toLowerCase;
```

**The Power of Lambda Expressions**

Java uses [functional interfaces](/techtalk/java/functional_interfaces) to represent lambda expression types. While it's best to use a built-in functional interface whenever possible, you sometimes need a custom functional interface.

To create your own functional interface, do two things:

* Annotate the interface with `@FunctionalInterface`, which is the Java 8 convention for custom functional interfaces.
* Ensure that the interface has just one abstract method.

Let's consider the following code:

```java
public interface StringAnalyzer {
  public Boolean analyze(String text, String keyword);
}
```

StringAnalyzer implementation in a traditional way could be:

```java
public class ContainsAnalyzer implements StringAnalyzer {

  public Boolean analyze(String text, String keyword){
    return text.contains(keyword);
  }

}
```

We can test code functionality as follow:

```java
@Test
public void shouldKnowContainKeyword(){
  StringAnalyzer analyzer = new ContainsAnalyzer();
  assertTrue(analyzer.analyze("In the end, it's not the years in your life that count. It's the life in your years", "life"));
}
```

Using functional interfaces could be:

```java
@FunctionalInterface
public interface StringAnalyzer {
  public Boolean analyze(String text, String keyword);
}
```

Test implementation:

```java
@Test
public void shouldKnowContainKeywordUsingLambda(){
  StringAnalyzer analyzer = (text, keyword) -> text.contains(keyword);
  assertTrue(analyzer.analyze("In the end, it's not the years in your life that count. It's the life in your years", "life"));
}
```

Since we are defining a interface `StringAnalyzer` we can use a lambda expression to implement this interface, that's it, you define or use a functional interface that accepts and returns exactly what you want. In this case we are replacing `ContainsAnalyzer` by a lambda expression.

Using lambda expressions we can create more expressive code like this one:

```java
@Test
public void shouldAnalyzeUsingContainsAndEndsWith(){
  StringAnalyzer analyzerContains = (text, keyword) -> text.contains(keyword);
  StringAnalyzer analyzerEndsWith = (text, keyword) -> text.endsWith(keyword);

  assertTrue(analyzerContains.analyze("In the end, it's not the years in your life that count. It's the life in your years", "life"));
  assertTrue(analyzerEndsWith.analyze("In the end, it's not the years in your life that count. It's the life in your years", "years"));
}
```

To download the code:

```bash
git clone https://github.com/josdem/java-topics.git
cd lambda-expressions/string-analyzer
```

[Return to the main article](/techtalk/java)
