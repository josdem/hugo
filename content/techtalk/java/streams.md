+++
title = "Streams"
categories = ["techtalk","code","java"]
tags = ["josdem","techtalks","programming","technology", "streams java", "java"]
date = "2017-07-02T10:13:38-05:00"
description = "Stream interface is defined in java.util.stream package. In Java 8, collections will start having methods that return Stream. Streams support Aggregate Operations. The common aggregate operations are filter, map, reduce, find, match and sort."
+++

Stream interface is defined in `java.util.stream package`. In Java 8, collections will start having methods that return Stream. Streams support Aggregate Operations. The common aggregate operations are filter, map, reduce, find, match and sort.

The `map` method is used to map each element to its corresponding result. The following Java code gets squares of numbers using map.

```java
import java.util.List;
import java.util.Arrays;
import java.util.ArrayList;
import java.util.stream.Collectors;

public class MapSquares {

  private List<Integer> square(List<Integer> numbers){
    return numbers.stream().map(it -> it * it).collect(Collectors.toList());
  }

  public static void main(String[] args){
    List<Integer> numbers = Arrays.asList(1,2,3,4,5,6,7,8);
    List<Integer> result = new MapSquares().square(numbers);
    assert result.size() == 8;
    assert result.get(1) == 4;
    assert result.get(3) == 16;
  }
}
```

The `sorted` method is used to sort a stream's elements.

```java
import java.util.List;
import java.util.Arrays;
import java.math.BigDecimal;
import java.util.stream.Collectors;

public class ListSorter {

	private List<BigDecimal> sort(List<BigDecimal> prices){
		return prices.stream().sorted( (p1, p2) -> p1.compareTo(p2) ).collect(Collectors.toList());
	}

	public static void main(String[] args){
		List<BigDecimal> prices = Arrays.asList(
			new BigDecimal("10.25"),
			new BigDecimal("25.90"),
			new BigDecimal("41.60"),
			new BigDecimal("9.50")
	  );

    List<BigDecimal> result = new ListSorter().sort(prices);
    assert 4 == result.size();
    assert new BigDecimal("9.50").equals(result.get(0));
    assert new BigDecimal("41.60").equals(result.get(3));
  }

}
```

The reduction operation combines all elements of the stream into a single result.

```java
import java.util.List;
import java.util.Arrays;
import java.util.Optional;

class StreamReductor {

  private Optional<String> getLargestName(List<String> nicknames){
    return nicknames.stream().reduce((s1,s2) -> s1.length() > s2.length() ? s1 : s2);
  }

  public static void main(String[] args){
    List<String> nicknames = Arrays.asList("josdem","tgrip","erich","martinvilegas","skuarch");
    Optional<String> result = new StreamReductor().getLargestName(nicknames);
    assert "martinvilegas" == result.get();
  }

}
```

`Stream.findFirst()` Get the first element of Stream in Java. Let's suppose that we have a List of String and we want to find out the first element which has the length greater than 10.

```java
import java.util.List;
import java.util.Arrays;
import java.util.Optional;

public class FirstFinder {

  private String findFirstLongName(List<String> nicknames){
    return nicknames.stream().filter(it -> it.length() > 10).findFirst().orElse("");
  }

  public static void main(String[] args){
    List<String> nicknames = Arrays.asList("josdem","tgrip","erich","martinvilegas","skuarch");
    String result = new FirstFinder().findFirstLongName(nicknames);
    assert "martinvilegas" == result;
  }
}
```

`allMatch`, `anyMatch` and `noneMatch` methods are applied on streams and matches the given Predicate, then returns boolean value.

```java
import java.util.List;
import java.util.Arrays;
import java.util.function.Predicate;

public class PersonMatcher {

  private Boolean anyPersonStartsWith(List<Person> persons, String prefix){
    Predicate<Person> predicate = person -> person.name.startsWith(prefix);
    return persons.stream().anyMatch(predicate);
  }

  private Boolean allAreSameType(List<Person> persons, RoleType type){
    Predicate<Person> predicate = person -> person.type == type;
    return persons.stream().allMatch(predicate);
  }

  private Boolean nonePersonEndsWith(List<Person> persons, String suffix){
    Predicate<Person> predicate = person -> person.name.endsWith(suffix);
    return persons.stream().noneMatch(predicate);
  }

  public static void main(String[] args){
    List<Person> persons = Arrays.asList(
      new Person("josdem", RoleType.DEVELOPER),
      new Person("tgrip", RoleType.DEVELOPER),
      new Person("skuarch", RoleType.DEVELOPER)
    );
    PersonMatcher matcher = new PersonMatcher();

    Boolean result = matcher.anyPersonStartsWith(persons, "j");
    assert result;

    result = matcher.allAreSameType(persons, RoleType.DEVELOPER);
    assert result;

    result = matcher.nonePersonEndsWith(persons, "zh");
    assert result;
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

Finally, let's review how to stream a file. Let's suppose you have a file with this information:

```
josdem
skuarch
edzero
heryxpc
jeduan
tgrip
```

And you want to get each line as a String element in a List. In Java 8, you can use `Files.lines` to read file as Stream.

```java
import java.util.List;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.Files;
import java.io.IOException;
import java.util.stream.Collectors;

public class FileStreamer {

  private List<String> read(Path path) throws IOException {
    return Files.lines(path).collect(Collectors.toList());
  }

  public static void main(String[] args) throws IOException {
    Path path = Paths.get("../resources/nicknames.txt");
    List<String> result = new FileStreamer().read(path);
    assert result.size() == 6;
    assert result.contains("josdem");
  }

}
```

`Stream.iterate` is used to iterate values from a seed to a value defined by an `UnaryOperator` if you want to know more about unary operator, please go to my previous post [Functional Interfaces](https://josdem.io/techtalk/java/functional_interfaces/#Basic_Functional_Interfaces)

```java
import java.util.List;
import java.util.ArrayList;
import java.util.stream.Stream;

public class StreamIterate {

    public static void main(String[] args){
        List<Integer> values = new ArrayList<>();

        Stream.iterate(0, i -> i + 2)
                .limit(6)
                .forEach(values::add);

        assert(values.size() == 6);
        assert(values.get(0) == 0);
        assert(values.get(1) == 2);
        assert(values.get(2) == 4);
        assert(values.get(3) == 6);
        assert(values.get(4) == 8);
        assert(values.get(5) == 10);
    }

}
```

To browse the code go [here](https://github.com/josdem/java-workshop), to download the code:

```bash
git clone https://github.com/josdem/java-workshop.git
cd streams/examples
```

To run the code:

```bash
javac ${JAVA_PROGRAM}.java
java -ea ${JAVA_PROGRAM}
```

[Return to the main article](/techtalk/java)

