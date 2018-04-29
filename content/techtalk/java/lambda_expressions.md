+++
tags = ["josdem","techtalks","programming","technology","java","lambda"]
categories = ["techtalk","code","java"]
date = "2016-12-17T10:05:56-06:00"
title = "Lambda Expressions"
description = "Lambda expression is an argument list follow by an arrow token and a body or code expression."
+++

Lambda expression is an argument list follow by an arrow token and a body or code expression.

```java
(x, y)  ->  x + y 
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

Let's consider the following code:

```java
@FunctionalInterface
public interface StringAnalyzer {
  public Boolean analyze(String text, String keyword);
}
```

StringAnalyzer implementation:

```java
public class ContainsAnalyzer implements StringAnalyzer {

  public Boolean analyze(String text, String keyword){
    return text.contains(keyword);
  }

}
```

Analyzer launcher:

```java
public class MainAnalyzer {

  public static void main(String[] args){
    StringAnalyzer analyzer = new ContainsAnalyzer();
    assert analyzer.analyze("In the end, it's not the years in your life that count. It's the life in your years", "life");
  }

}
```

Since we are defining a interface `StringAnalyzer` we can use a lambda expression to implement this interface, that's it, you define or use a functional interface that accepts and returns exactly what you want. In this case we are replacing `ContainsAnalyzer` by a lambda expression.

```java
public class MainAnalyzer {

  public static void main(String[] args){
    StringAnalyzer analyzer = (text, keyword) -> text.contains(keyword);
    assert analyzer.analyze("In the end, it's not the years in your life that count. It's the life in your years", "life");
  }

}
```

Lambda expressions also can be treated like a variables, it could be assigned, it could be pass as parameter, so therefore the code is easily reuse. Let's create a new class so you can see how we can use lambda as parameters:

```java
public class AnalyzerTool {

  public Boolean analyze(String text, String keyword, StringAnalyzer analizer){
    return analizer.analyze(text, keyword);
  }

}
```

So now we can pass an StringAnalyzer as parameter which could be a lambda expression

```java
public class MainAnalyzer {

  public static void main(String[] args){
    StringAnalyzer analyzerContains = (text, keyword) -> text.contains(keyword);
    StringAnalyzer analyzerEndsWith = (text, keyword) -> text.endsWith(keyword);

    assert analyzerContains.analyze("In the end, it's not the years in your life that count. It's the life in your years", "life");
    assert analyzerEndsWith.analyze("In the end, it's not the years in your life that count. It's the life in your years", "years");

  }
}
```

To download the code:

```bash
git clone https://github.com/josdem/java-topics.git
cd lambda-expressions/string-analyzer
```

[Return to the main article](/techtalk/java)
