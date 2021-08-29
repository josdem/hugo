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
* [Min-Max Sum](#Min_Max_Sum)
* [Birthday Cake Candles](#Birthday_Cake_Candles)
* [String Compressor](#String_Compressor)
* [Sock Merchant](#Sock_Merchant)
* [Electronics Shop](#Electronics_Shop)
* [Square my List](#Square_my_List)


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
import java.util.ArrayList;
import java.util.Arrays;
import java.util.stream.Collectors;

public class CommonElementsFinder {

  private List<Integer> find(List<Integer> first, List<Integer> second){
    return first.stream().filter(second::contains).collect(Collectors.toList());
  }

  public static void main(String[] args){
    List<Integer> first = Arrays.asList(1, 2, 3, 4, 5);
    List<Integer> second = Arrays.asList(1, 3, 5, 7, 9);
    List<Integer> result = new CommonElementsFinder().find(first, second);
    assert new ArrayList<Integer>(Arrays.asList(1,3,5)).equals(result);
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

<a name="Min_Max_Sum">
## Min-Max Sum
</a>

Given five positive integers, find the minimum and maximum values that can be calculated by summing exactly four of the five integers. Then get the respective minimum and maximum values.

**Output**

* A integer denoting minimum value that can be calculated by summing exactly four of the five integers.
* A integer denoting maximum value that can be calculated by summing exactly four of the five integers.

**Sample Input**

```bash
[1,2,3,4,5]
```

**Sample Output**

```bash
10 14
```

**Explanation**

Our initial numbers are `1, 2, 3, 4` and `5`. We can calculate the following sums using four of the five integers:

* If we sum everything except `1`, our sum is `2 + 3 + 4 + 5 = 14`.
* If we sum everything except `2`, our sum is `1 + 3 + 4 + 5 = 13`.
* If we sum everything except `3`, our sum is `1 + 2 + 4 + 5 = 12`.
* If we sum everything except `4`, our sum is `1 + 2 + 3 + 5 = 11`.
* If we sum everything except `5`, our sum is `1 + 2 + 3 + 4 = 10`.

As you can see, the minimal sum is `10` and the maximal sum is `14`.

**Solution**

```java
import java.util.List;
import java.util.Arrays;
import java.util.SortedSet;
import java.util.TreeSet;

public class MinMaxFinder {

  private SortedSet<Integer> find(List<Integer> numbers){
    SortedSet<Integer> collection = new TreeSet<Integer>();
    for(int i=0; i<numbers.size(); i++){
      collection.add(numbers.stream().mapToInt(it -> it).sum() - numbers.get(i));
    }
    return collection;
  }

  public static void main(String[] args){
    List<Integer> numbers = Arrays.asList(1,2,3,4,5);
    SortedSet<Integer> result = new MinMaxFinder().find(numbers);
    assert 10 == result.first();
    assert 14 == result.last();
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

Given a String "aaabbbbcc" compress it in such a way that you get pairs: character, count as: a3b4c2.

**Constraints**

* `$a \le character \le z$`
* `$1 \le string(length) \le 1000$`

**Sample Input**

```bash
String = "aaabbbbcc"
```

**Sample Output**

```bash
a3b4c2
```

**Solution**

```java
import java.util.Map;
import java.util.stream.Collectors;

public class StringCompressor {

  private String compress(String string){
    Map<Character, Long> map = string.chars()
        .mapToObj(i -> (char)i)
        .collect(Collectors.groupingBy(it -> it, Collectors.counting()));

    StringBuffer sb = new StringBuffer();

    map.forEach((k,v) -> sb.append(k.toString() + v.toString()));
    return sb.toString();
  }

  public static void main(String[] args){
    String string = "aaabbbbcc";
    String result = new StringCompressor().compress(string);
    System.out.println(result);
  }

}
```

<a name="Sock_Merchant">
## Sock Merchant
</a>

John works at a clothing store and he's going through a pile of socks to find the number of matching pairs. More specifically, he has a pile of `$N$` loose socks where each sock `$i$`  is labeled with an integer, `$Ci$`, denoting its color. He wants to sell as many socks as possible, but his customers will only buy them in matching pairs. Two socks, `$i, j$`, are a single matching pair if they have the same color `$(Ci == Cj)$`.

Given `$N$` and the color of each sock, how many pairs of socks can John sell?

**Input Format**

An integers collection describing the respective values of `$c0, c1, c2,..., cN-1$`.

**Constraints**

* `$1 \le N \le 100$`
* `$1 \le Ci \le 100$`

**Output Format**

Total number of matching pairs of socks that John can sell.

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
import java.util.Map;
import java.util.List;
import java.util.Arrays;
import java.util.ArrayList;
import java.util.stream.Collectors;

public class SockPairMatcher{

  private Integer match(List<Integer> colors){
    Map<Integer, Long> map = colors.stream()
      .collect(Collectors.groupingBy(it->it, Collectors.counting()));

    List<Long> values = map.entrySet().stream()
      .filter(it -> it.getValue() / 2 > 0)
      .map(Map.Entry::getValue)
      .collect(Collectors.toList());

    Long result = values.stream()
      .mapToLong(it -> it / 2)
      .sum();

    return result.intValue();
  }

  public static void main(String[] args){
    List<Integer> colors = Arrays.asList(10, 20, 20, 10, 10, 30, 50, 10, 20);
    Integer result = new SockPairMatcher().match(colors);
    assert 3 == result;
  }

}
```

We are solving this challenge in three steps:

1. We are grouping by occurrences
2. From a map we are getting `Map.Entry` collection as stream, then we filtered every item value by which one is divisible by 2, finally we stored filtered values as a list.
3. We are counting how many pairs we can create by dividing every item from long collection by 2

<a name="Electronics_Shop">
## Electronics Shop
</a>

Monica wants to buy exactly one keyboard and one USB drive from her favorite electronics store. The store sells `$N$` different brands of keyboards and `$M$` different brands of USB drives. Monica has exactly `$S$` dollars to spend, and she wants to spend as much of it as possible (i.e., the total cost of her purchase must be maximal).

Given the price lists for the store's keyboards and USB drives, find and print the amount of money Monica will spend. If she doesn't have enough money to buy one keyboard and one USB drive, print `$-1$` instead.

**Note:** She will never buy more than one keyboard and one USB drive even if she has the leftover money to do so.

**Input Format**

* Amount of money Monica has to spend in electronics.
* Integer collection denoting the prices of each keyboard brand.
* Integer collection denoting the prices of each USB drive brand.

**Constraints**

* `$1 \le N,M \le 1000$`
* `$1 \le S \le 10^6$`
* `$1 \le price \le 10^6$`

**Output Format**

Single integer denoting the amount of money Monica will spend. If she doesn't have enough money to buy one keyboard and one USB drive `-1` instead.

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
import java.util.Map;
import java.util.List;
import java.util.Arrays;
import java.util.ArrayList;
import java.util.Optional;
import java.util.AbstractMap.SimpleEntry;

import java.util.stream.Collectors;

public class ShopCalculator {

  private Integer calculate(final Integer amount, List<Integer> keyboards, List<Integer> usbs){
    List<Map.Entry<Integer,Integer>> pairList = new ArrayList<Map.Entry<Integer,Integer>>();

    keyboards.forEach( k ->
      usbs.forEach( u ->
        pairList.add(new SimpleEntry<Integer,Integer>(k,u))
      )
    );

    List<Integer> electronicsCost = pairList.stream().map(entry -> entry.getKey() + entry.getValue()).collect(Collectors.toList());

    Optional<Integer> result = electronicsCost.stream().filter(it -> it <= amount).max(Integer::compare);

    return result.isPresent() ? result.get() : -1;
  }

  public static void main(String[] args){
    List<Integer> keyboards = Arrays.asList(3, 1);
    List<Integer> usbs = Arrays.asList(5, 2, 8);
    Integer amount = 10;
    Integer result = new ShopCalculator().calculate(amount, keyboards, usbs);
    assert 9 == result;
  }

}
```

I solved this challenge in three steps:

1. We are creating keyboard and usb price combination using a `Map.Entry` pair object
2. We are summarizing those price combinations and stored as a `List`
3. We are getting max result combination comparing with equals or less that our amount

<a name="Square_my_List">
## Square my List
</a>

Given a list of numeric elements, square every element from the list and retrun the result in another list collection.

**Solution**

```java
import java.util.List;
import java.util.Arrays;
import java.util.stream.Collectors;

public class SquareCalculator {

  private List<Integer> square(List<Integer> numbers){
    return numbers.stream()
      .map(i -> i * i)
      .collect(Collectors.toList());
  }

  public static void main(String[] args){
    List<Integer> numbers = Arrays.asList(1, 2, 3, 7, 9, 12, 15);
    List<Integer> result = new SquareCalculator().square(numbers);
    assert result.get(0) == 1;
    assert result.get(2) == 9;
    assert result.get(4) == 81;
    assert result.get(6) == 225;
  }

}
```

To browse the project go [here](https://github.com/josdem/algorithms-workshop), to download the project:

```bash
git clone git@github.com:josdem/algorithms-workshop.git
cd simple-algorithms
```

To run Java code:

```bash
javac ${PROGRAM_NAME}.java
java -ea ${PROGRAM_NAME}
```

To run Groovy code:

```bash
groovy ${PROGRAM_NAME}.groovy
```


[Return to the main article](/techtalk/algorithms)
