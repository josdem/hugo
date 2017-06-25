+++
categories = ["techtalk","code"]
tags = ["josdem","techtalks","programming","technology", "Java 8", "streams java", "collectors java"]
date = "2017-06-25T10:48:05-05:00"
title = "Stream Collectors"

+++

This time I will show you how to use collectors over streams to group by, concatenate, map and list.

Convert list elements to a string separated by ','

```java
import java.util.List;
import java.util.Arrays;
import java.util.stream.Collectors;

public class StreamToStringConverter {

  private String parse(){
    return Arrays.asList("Java", "C++", "Lisp", "Haskell").
      stream().collect(Collectors.joining(","));
  }

  public static void main(String[] args){
    String result = new StreamToStringConverter().parse();
    assert "Java,C++,Lisp,Haskell".equals(result);
  }

}
```

From a list, generate another list where elements length is equals to four.

```java
import java.util.List;
import java.util.Arrays;
import java.util.stream.Collectors;

public class StreamToListConverter {

  private List<String> parse(){
    return Arrays.asList("Java", "C++", "Lisp", "Haskell").
      stream().filter( it -> it.length() == 4 ).collect(Collectors.toList());  // 1
  }

  public static void main(String[] args){
    List<String> result = new StreamToListConverter().parse();
    assert 2 == result.size();
    assert result.contains("Java");
    assert result.contains("Lisp");
  }

}
```

(1) We can use also `Collectors.toSet()`

From a list of strings get a map with string element as key and string length as value

```java
import java.util.Map;
import java.util.Arrays;
import java.util.function.Function;
import java.util.stream.Collectors;

public class StreamToMapConverter {

  private Map<String, Integer> parse(){
    return Arrays.asList("Java", "C++", "Lisp", "Haskell").
      stream().collect(Collectors.toMap(Function.identity(), String::length));
  }

  public static void main(String[] args){
    Map<String, Integer> result = new StreamToMapConverter().parse();
    assert 4 == result.size();
    assert result.get("Java") == 4;
    assert result.get("C++") == 3;
    assert result.get("Lisp") == 4;
    assert result.get("Haskell") == 7;
  }

}
```

`Function.identity()` is just a shortcut for defining function that accepts and return the same value;

`String::length` is a shortcut to define object type and method to return

From a list of persons, group by role

```java
import java.util.List;
import java.util.Map;
import java.util.Arrays;
import java.util.stream.Collectors;

public class GroupByCollector {

  private Map<RoleType, List<Person>> parse(List<Person> persons){
    return persons.stream().collect(Collectors.groupingBy(Person::getType));
  }

  public static void main(String[] args){
    List<Person> persons = Arrays.asList(
      new Person("josdem", RoleType.DEVELOPER),
      new Person("tgtip", RoleType.DEVELOPER),
      new Person("erich", RoleType.TESTER)
    );
    Map<RoleType, List<Person>> result = new GroupByCollector().parse(persons);
    assert result.get(RoleType.DEVELOPER).size() == 2;
    assert result.get(RoleType.TESTER).size() == 1;
  }

}

class Person {
  String name;
  RoleType type;

  public Person(String name, RoleType type){
    this.name = name;
    this.type = type;
  }

  RoleType getType(){
    return type;
  }

}

enum RoleType {
  DEVELOPER, TESTER
}
```

To download the code:

```bash
git clone https://github.com/josdem/java-topics.git
cd streams/collect
```

To run the code:

```bash
javac ${JAVA_PROGRAM}.java
java -ea ${JAVA_PROGRAM}
```

[Return to the main article](/techtalk/java)
