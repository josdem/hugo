+++
date = "2017-06-21T15:27:41-05:00"
title = "Stream Filters"
tags = ["josdem","techtalks","programming","technology", "Java 8", "streams java", "filters java"]
categories = ["techtalk","code"]

+++

A stream represents a sequence of elements and supports different operations to perform calculations in those elements:

```java
import java.util.List;
import java.util.Arrays;
import java.util.stream.Collectors;
import java.util.function.Predicate;

public class StringFilter {

  private List<String> parse(){
    return Arrays.asList("Java", "C++", "Lisp", "Haskell").
      stream().filter( p -> p.startsWith("J")).               // 1
      collect(Collectors.<String>toList());
  }

  public static void main(String[] args){
    List<String> result = new StringFilter().parse();
    assert 1 == result.size();
    assert "Java" == result.get(0);
  }

}
```

1. After creating a Stream of Strings we filtered by those starts with letter J.

If we want to filter a list, array or map based in any condition, we can do it easily using lambda expression with stream `filter()` method.

Here we are filtering a list and then counting the number of elements.


```java
import java.util.List;
import java.util.Arrays;
import java.util.stream.Collectors;

public class CountFilter {

  private Long parse(){
    return Arrays.asList("Java", "C++", "Lisp", "Haskell").
      stream().filter( p -> p.length() == 4).
      collect(Collectors.counting());
  }

  public static void main(String[] args){
    Long result = new CountFilter().parse();
    assert 2L == result;
  }

}
```

Let's review an example using numbers, we want to find the max value from a list.


```java
import java.util.Arrays;
import java.util.OptionalInt;

public class MaxInteger {

  private OptionalInt parse(){
    return Arrays.asList(3, 13, 31, 35, 41, 50, 66, 79, 100).
      stream().mapToInt(Integer::intValue).max();
  }

  public static void main(String[] args){
    OptionalInt result = new MaxInteger().parse();
    assert 100 == result.getAsInt();
  }

}
```

Here we will filter our list for a range between 20 and 60 and then we will calculate the sum.

```java
import java.util.List;
import java.util.Arrays;

public class IntegerFilter {

  private Integer parse(){
    return Arrays.asList(3, 13, 31, 35, 41, 50, 66, 79, 100).
      stream().filter( n -> n >= 20 && n <= 60 ).mapToInt(Integer::intValue).sum();
  }

  public static void main(String[] args){
    Integer result = new IntegerFilter().parse();
    assert 157 == result;
  }

}
```

To download the code:

```bash
git clone https://github.com/josdem/java-topics.git
cd streams/filters
```

To run the code:

```bash
javac ${JAVA_PROGRAM}.java
java -ea ${JAVA_PROGRAM}
```

[Return to the main article](/techtalk/java)
