+++
tags = [
  "josdem",
  "techtalks",
  "programming",
  "technology",
  "java",
  "lambda"
]
categories = [
  "techtalk",
  "code",
]
date = "2016-12-17T10:05:56-06:00"
title = "Lambda Expressions"

+++

Lambda expression is an argument list follow by an arrow token and a body or code expression.

|Argument List   | Arrow Token  | Body |
|---|---|---|
| (int x, int y) | -> | x + y |

**Basic Lambda examples**

```java
(Integer x, Integer y) -> x + y
(String s) -> s.contains("word")
```
Let's consider the following code:

```java
public interface StringAnalyzer {
  public Boolean analyze(String text, String keyword);
}
```

```java
public class ContainsAnalyzer implements StringAnalyzer {

  public Boolean analyze(String text, String keyword){
    return text.contains(keyword);
  }

}
```

```java
public class MainAnalyzer {

  public static void main(String[] args){
    StringAnalyzer analyzer = new ContainsAnalyzer();
    assert analyzer.analyze("In the end, it's not the years in your life that count. It's the life in your years", "life");
  }

}
```

Since we are defining a interface `StringAnalyzer` we can use a lambda expression to implement this interface, that's it, you define or use a functional interface that accepts and returns exactly what you want.

```java
public class MainAnalyzer {

  public static void main(String[] args){
    StringAnalyzer analyzer = (String text, String keyword) -> text.contains(keyword);
    assert analyzer.analyze("In the end, it's not the years in your life that count. It's the life in your years", "life");
  }

}
```

Lambda expressions also can be treated like a variables, it could be assigned, it could be pass as parameter, so therefore the code is easily reuse.

To download the code:

```bash
git clone https://github.com/josdem/java-topics.git
cd lambda/string-analyzer
```

[Return to the main article](/techtalk/java)
