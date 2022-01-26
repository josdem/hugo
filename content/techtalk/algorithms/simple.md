+++
categories = ["algorithms", "pseudocode"]
date = "2016-07-04T21:00:30-05:00"
tags = ["josdem", "techtalks", "algorithms", "power set", "palindrome"]
title = "Algorithms"
description = "This section is about solving simple algorithms, coding challenges, puzzles, katas and dojos in Java and Groovy."
+++

This section is about solving simple algorithms, coding challenges, puzzles, katas and dojo material in Java.

* [Sum a Collection](#Sum_a_Collection)
* [Biggest Number](#Biggest_Number)
* [Most Popular in the array](#Most_Popular_in_the_Array)
* [Common Elements in two Collections](#Common_Elements_in_two_Collections)
* [Number Counter](#Number_Counter)
* [Characters Counter](#Characters_Counter)
* [Birthday Cake Candles](#Birthday_Cake_Candles)
* [String Compressor](#String_Compressor)
* [Sock Merchant](#Sock_Merchant)
* [Smallest Biggest](#Smallest_Biggest)
* [Electronics Shop](#Electronics_Shop)
* [Square my List](#Square_my_List)
* [Digital Adder](#Digital_Adder)


<a name="Sum_a_Collection">
## Sum a Collection
</a>

Given an array of integers, find the sum of its elements.

```java
import java.util.List;

public class CollectionAdder {

    public int sum(List<Integer> numbers) {
        return numbers.stream().reduce(0, Integer::sum);
    }
}
```

<a name="Biggest_Number">
## Biggest Number
</a>

Messages with random data are coming! But we just care about numbers!
Your task is to implement a function which removes all non numeric data and return just the biggest number messages = `"hi", "2", "@#$%", "32"` result = 32


```java
import java.util.List;

public class BiggestNumberFinder {

  private String regex = "-?[0-9]+.?[0-9]+";

  public double find(List<String> numbers) {
    return numbers.stream()
        .filter(it -> it.matches(regex))
        .map(it -> Double.parseDouble(it))
        .max(Double::compare)
        .get();
  }
}
```

<a name="Most_Popular_in_the_Array">
## Most Popular in the Array
</a>

Given a collection with numbers: `34 , 31, 34, 56, 12, 35, 24, 34, 69, 18` Write a function that finds most popular number in the array, in this case 34 because it is the number that appears most often.

```java
import java.util.Comparator;
import java.util.List;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;

public class PopularFinder {

  public int find(List<Integer> numbers) {
    return numbers.stream()
        .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()))
        .entrySet()
        .stream()
        .max(Comparator.comparing(Map.Entry::getValue))
        .get()
        .getKey();
  }
}
```

<a name="Common_Elements_in_two_Collections">
## Common Elements in two Collections
</a>

I have two arrays with integers. I want to return elements in common.

**Example**

```bash
first = [1,2,3,4,5]
second = [1,3,5,7,9]
result = [1,3,5]
```

**Solution**

```java
import java.util.List;
import java.util.stream.Collectors;

public class CommonElements {

  public List<Integer> find(List<Integer> first, List<Integer> second) {
    return first.stream().filter(it -> second.contains(it)).collect(Collectors.toList());
  }
}
```

<a name="Number_Counter">
## Number Counter
</a>

Given an integer collection, return an array with three elements:
how many of them are positive, how many negative and how many are zeros.

**Sample Input**

```bash
[-4, 3, -9, 0, 4, 1]
```

**Expected Output**

```bash
[3, 2, 1]
```

**Solution**

```java
import java.util.Arrays;

public class NumbersCounter {

    public int[] count(int[] numbers) {
        return new int[]{
                (int)Arrays.stream(numbers).filter(n -> n > 0).count(),
                (int)Arrays.stream(numbers).filter(n -> n < 0).count(),
                (int)Arrays.stream(numbers).filter(n -> n == 0).count()
        };
    }
}
```

<a name="Characters_Counter">
## Characters Counter
</a>

Create two functions one for counting vowels and another for counting consonants. Given a string, when that string is josdem, then counting vowels should be 2 and consonants 4.

**Solution**

```java
import java.util.Arrays;
import java.util.List;

public class CharactersCounter {
  private List<Character> vowels = Arrays.asList('a', 'e', 'i', 'o', 'u');
  private List<Character> consonants =
      Arrays.asList(
          'b', 'c', 'd', 'f', 'g', 'h', 'j', 'k', 'l', 'm', 'n', 'r', 'p', 'q', 's', 't', 'v', 'w',
          'x', 'y', 'z');

  public int countVowels(String keyword) {
    return (int) keyword.chars().filter(ch -> vowels.contains((char) ch)).count();
  }

  public int countConsonants(String keyword) {
    return (int) keyword.chars().filter(ch -> consonants.contains((char) ch)).count();
  }
}
```

<a name="Birthday_Cake_Candles">
## Birthday Cake Candles
</a>

It is your birthday! And you want to make it special, so you want to keep only biggest candles in the cake
Your task is to create a function that removes all small candles and just keep the biggest ones.

**Sample Input**

```bash
[3,2,1,3]
```

**Sample Output**

```bash
[3,3]
```

**Solution**

```java
import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

public class CandleCounter {

  public List<Integer> count(List<Integer> candles) {
    Optional<Integer> biggest = candles.stream().max(Integer::compare);
    return candles.stream().filter(it -> it == biggest.get()).collect(Collectors.toList());
  }
}
```

<a name="String_Compressor">
## String Compressor
</a>

Given a String "aaabbbbcc" when we call compress method then we have a compressed string like: "a3b4c2"

**Sample Input**

```bash
String = "aaabbbbcc"
```

**Sample Output**

```bash
"a3b4c2"
```

**Solution**

```java
import java.util.Map;
import java.util.stream.Collectors;

public class StringCompressor {

  public String compress(String keyword) {
    Map<Object, Long> map =
        keyword
            .chars()
            .mapToObj(it -> (char) it)
            .collect(Collectors.groupingBy(it -> it, Collectors.counting()));
    StringBuilder sb = new StringBuilder();
    map.forEach(
        (k, v) -> {
          sb.append(k);
          sb.append(v);
        });
    return sb.toString();
  }
}
```

<a name="Sock_Merchant">
## Sock Merchant
</a>

John works at a clothing store and he's going through a pile of socks to find the number of matching pairs. Given a collection with socks' colors `10, 20, 20, 10, 10, 30, 50, 10, 20` write a function to find out how many pairs you can get.

**Sample Input**

```bash
colors = [10, 20, 20, 10, 10, 30, 50, 10, 20]
```

**Sample Output**

```bash
3
```

**Explanation**

<img src="/img/techtalks/algorithms/sock_merchant.png"/>

As you can see from the figure above, we can match three pairs of socks.

**Solution**

```java
import java.util.List;
import java.util.Map;
import java.util.function.Function;
import java.util.stream.Collectors;

class SockPairFinder {

  public Integer findPairsBy(List<Integer> colors) {
    Map<Integer, Long> map =
        colors.stream().collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
    List<Long> values =
        map.entrySet().stream()
            .map(Map.Entry::getValue)
            .filter(it -> it / 2 > 0)
            .collect(Collectors.toList());
    Long result = values.stream().mapToLong(it -> it / 2).sum();
    return result.intValue();
  }
}
```

<a name="Smallest_Biggest">
## Smallest Biggest
</a>

Find smaller and biggest numbers in a collection, then return another collection with result. Given a collection like: `7, 5, 2, 4, 3, 9` when I call find method then I will get a collection with `2, 9` values.

**Sample Input**

```bash
numbers = [7, 5, 2, 4, 3, 9]
```

**Sample Output**

```bash
result = [2, 9]
```

**Solution**

```java
import java.util.Arrays;
import java.util.List;

public class SmallerBiggestFinder {
  public List<Integer> find(List<Integer> numbers) {
    Integer smaller = numbers.stream().reduce((a, b) -> a < b ? a : b).get();
    Integer biggest = numbers.stream().reduce((a, b) -> a > b ? a : b).get();
    return Arrays.asList(smaller, biggest);
  }
}
```

<a name="Electronics_Shop">
## Electronics Shop
</a>

Monica wants to buy exactly one keyboard and one USB drive from her favorite electronics store. The store sells N different brands of keyboards and M different brands of USB drives. Monica has exactly S dollars to spend, and she wants to spend as much of it as possible (i.e., the total cost of her purchase must be maximal).
Given the amount of money Monica has to spend in electronics and the prices lists for the store's keyboards and USB drives, find out the amount of money Monica will spend.

**Sample Input**

```bash
keyboards = [3, 1]
usbs = [5, 2, 8]
amount = 10
```

**Sample Output**

```bash
9
```

**Explanation**

She can buy the `$2nd$` keyboard and the `$3rd$` USB drive for a total cost of `$8 + 1 = 9$`.

**Solution**

```java
import java.util.AbstractMap.SimpleEntry;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class ShopCalculator {

  public int compute(Integer amount, List<Integer> keyboards, List<Integer> usbs) {
    final List<Map.Entry<Integer, Integer>> pairs = new ArrayList<>();

    keyboards.forEach(
        k ->
            usbs.forEach(
                u -> {
                  pairs.add(new SimpleEntry<>(k, u));
                }));

    final List<Integer> prices =
        pairs.stream().map(entry -> entry.getKey() + entry.getValue()).collect(Collectors.toList());
    return prices.stream().filter(it -> it <= amount).max(Integer::compare).get();
  }
}
```

<a name="Square_my_List">
## Square my List
</a>

Given a numeric list elements, square every element from the list and retrun the result in another collection.

**Sample Input**

```bash
numbers = [15, 20, 4, 8]
```

**Sample Output**

```bash
expectedCollection = [6, 2, 4, 8]
```


**Solution**

```java
import java.util.List;
import java.util.stream.Collectors;

public class SquareCalculator {
  public List<Integer> square(List<Integer> numbers) {
    return numbers.stream().map(it -> it * it).collect(Collectors.toList());
  }
}
```

<a name="Digital_Adder">
## Digital Adder
</a>

Given a numeric collection, when I call add method, then I will get a collection with every element summing its digits

**Solution**

```java
import java.util.List;
import java.util.stream.Collectors;

public class DigitalAdder {
  public List<Integer> add(List<Integer> numbers) {
    List<String> numbersAsString =
        numbers.stream().map(it -> String.valueOf(it)).collect(Collectors.toList());
    return numbersAsString.stream()
        .map(string -> string.chars().map(ch -> Integer.valueOf(Character.toString(ch))).sum())
        .collect(Collectors.toList());
  }
}
```


To browse the project go [here](https://github.com/josdem/algorithms-workshop), to download the project:

```bash
git clone git@github.com:josdem/algorithms-workshop.git
cd simple-algorithms
```

To run Java project code: Go to the kata directory and do

```bash
gradle test
```


[Return to the main article](/techtalk/algorithms)
