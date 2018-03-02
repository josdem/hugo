+++
categories = ["algorithms", "pseudocode"]
date = "2016-07-04T21:00:30-05:00"
tags = ["josdem", "techtalks", "algorithms", "power set", "palindrome"]
title = "Algorithms"

+++

This section is about solving simple algorithms, coding challenges, puzzles, katas and dojos in Java and Groovy.

* [Find Palindrome](#Palindrome)
* [Biggest Number](#Biggest_Number)
* [Sum a Collection](#Sum_a_Collection)
* [Staircase](#Staircase)
* [Most Popular in the array](#Most_Popular_in_the_Array)
* [Center Point in 2d Array](#Center_Point_in_2d_Array)
* [Power Set](#Power_Set)
* [Collection Adder](#Collection_Adder)
* [Common Elements in two Collections](#Common_Elements_in_two_Collections)
* [Plus Minus](#Plus_Minus)
* [Min-Max Sum](#Min_Max_Sum)
* [Birthday Cake Candles](#Birthday_Cake_Candles)
* [Breaking the Records](#Breaking_Records)
* [Birthday Chocolate](#Birthday_Chocolate)
* [String Compressor](#String_Compressor)
* [Bon Appetit](#Bon_Appetit)

<a name="Palindrome">
## Palindrome
</a>

A Palindrome is a word phrase, or number that reads the same backward or forward.

```groovy
Boolean isPalindrome(String string){
  string.reverse().equals(string)
}

assert isPalindrome('anitalavalatina')
```

<a name="Biggest_Number">
## Biggest Number
</a>

From a string list with character, number pair elements, extract number and get the biggest

example: "a1", "b2", "c3" biggest: 3

```java
import java.util.OptionalInt;
import java.util.Arrays;
import java.util.List;

public class BiggestNumber {

  private OptionalInt find(List<String> strings){
    return strings.stream().map(s -> s.substring(1))
           .mapToInt(Integer::parseInt)
           .max();
  }

  public static void main(String[] args){
    List<String> strings = Arrays.asList("a1","b2","c3");
    OptionalInt result = new BiggestNumber().find(strings);
    assert result.getAsInt() == 3;
  }

}
```

<a name="Sum_a_Collection">
## Sum a Collection
</a>

Given an array of integers, find the sum of its elements.

**Groovy Version**

```groovy
def array = [1, 2, 3, 4, 10, 11]
println array.sum{it}
```


**Java Version**

```java
import java.util.List;
import java.util.Arrays;
import java.util.stream.IntStream;

public class CollectionSum {

  private Integer sum(List<Integer> array){
    return array.stream().mapToInt(it -> it).sum();
  }

  public static void main(String[] args){
    List<Integer> array = Arrays.asList(1,2,3,4,10,11);
    Integer result = new CollectionSum().sum(array);
    System.out.println(result);
  }

}
```

<a name="Staircase">
## Staircase
</a>

Consider a staircase of size 4:

```
#
##
###
####
```

Observe that its base and height are both equal. Write a program that prints a staircase of size `n`.

```java
public class Staircase {

  private void printStair(Integer size){
    for(int j=1; j<=size; j++){
      for(int i=1; i<=j; i++){
        System.out.print("#");
      }
      System.out.print("\n");
    }
  }

  public static void main(String[] args){
    Integer size = 4;
    new Staircase().printStair(size);
  }

}
```

<a name="Most_Popular_in_the_Array">
## Most Popular in the Array
</a>

Assume I have an array that looks like the following:

```groovy
def array = [34,31,34,56,12,35,24,34,69,18]
```

Write a function that can determine what is the most popular in the array, in this case "34" because it is the number that appears the most often.

**Groovy Version**

```groovy
Integer mostPopular(def array){
  Integer occurrences = 0
  Integer mostPopular = 0
  array.each{
    if(array.count { it } > occurrences){
      occurrences = array.count { it }
      mostPopular = it
    }
  }
  mostPopular
}
```

**Java Version**

```java
import java.util.Map;
import java.util.List;
import java.util.Arrays;
import java.util.Comparator;
import java.util.stream.Collectors;

public class PopularDetector {

  private Integer find(List<Integer> numbers){
     Integer value = numbers.stream()
       .collect(Collectors.groupingBy(it -> it, Collectors.counting()))
       .entrySet()                                     // 1
       .stream()
       .max(Comparator.comparing(Map.Entry::getValue))
       .get()                                          // 2
       .getKey();
     return value;
  }


  public static void main(String[] args){
    List<Integer> numbers = Arrays.asList(34,31,34,56,12,35,24,34,69,18);
    Integer result = new PopularDetector().find(numbers);
    assert result == 34;
  }

}
```

1. Gets Map.Entry interface
2. Gets entry resultant from max comparator


<a name="Center_Point_in_2d_Array">
## Center Point in 2d Array
</a>

Compute average all the x, y coordinates and find the location in the dead center of them.

```groovy
class Point{
  Integer x
  Integer y
}

def computeAverage(List<Point> points){
  Float totalX = 0;
  Float totalY = 0;

  points.each{
    totalX += it.x
    totalY += it.y
  }
  [totalX/points.size(),totalY/points.size()]
}

computeAverage([new Point(x:3,y:4), new Point(x:2,y:1)])
```

Output:

```
[2.5, 2.5]
```

<a name="Power_Set">
## Power Set
</a>

**Problem**

Solve power set algorithm. Power set is the set of all subsets of a set. For example, the power set of the set {a, b, c} consists of the sets:

```
{}
{a}
{b}
{c}
{a,b}
{a,c}
{b,c}
{a,b,c}
```

Note that:

* The empty set {} is in the power set
* The set itself is in the power set

**Solution**

A subset can be represented as an array of boolean values of the same size as the set, called a characteristic vector. Each boolean value indicates whether the corresponding element in the set is present or absent in the subset.

This gives following correspondences for the set {a, b, c}:

```
[0,0,0] = {}
[1,0,0] = {a}
[0,1,0] = {b}
[0,0,1] = {c}
[1,1,0] = {a,b}
[1,0,1] = {a,c}
[0,1,1] = {b,c}
[1,1,1] = {a,b,c}
```

```groovy
def array = ['a','b','c']

def computeSubsets(def array){
  def matcher = [[0,0,0],[1,0,0],[0,1,0],[0,0,1],[1,1,0],[1,0,1],[0,1,1],[1,1,1]]
  def all = []
  matcher.each { col ->
    def subset = []
    col.eachWithIndex { row, i ->
      if(row == 1){
        subset << array[i]
      }
    }
    if(!subset.isEmpty())
      all << subset
  }
  all.add(0,[])
  all
}

computeSubsets(array)

```

Output

```
[[], [a], [b], [c], [a, b], [a, c], [b, c], [a, b, c]]
```

<a name="Collection_Adder">
## Collection Adder
</a>

**Problem**

You have two arrays, sum each element from right to left following same rules we know about sum.

**Solution**

1. If first collection and second collection size is zero then return empty list
2. Iterate every collection from right to left sum each element and storing carrier as local variable
3. Store the resultant value in the specific index in the resultant array list

**Example**

```bash
[1,0] => 10
[  5] =>  5
------------
[1,5]
```

```java
import java.util.List;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.stream.Collectors;

public class CollectionAdder {

  List<Integer> integerAdder(int[] first, int[] second) {
    int carrier = 0;
    List<Integer> result = new ArrayList<Integer>();

    if(first.length == 0 && second.length == 0){
      return new ArrayList<Integer>();
    }

    int j = second.length-1;
    for(int i=first.length-1; i>=0; i--){
      int valueA = first[i];
      int valueB = 0;

      if(j>=0){
        valueB = second[j];
      }

      int parcialValue = valueA + valueB + carrier;
      int mod = parcialValue % 10;
      result.add(0,mod);

      carrier = parcialValue / 10;
      j--;
    }

    if(carrier > 0){
      result.add(0,carrier);
    }

    return result;
  }


  public static void main(String[] args){
    int[] first = {1,0};
    int[] second = {5};
    List<Integer> result = new CollectionAdder().integerAdder(first, second);
    assert new ArrayList<Integer>(Arrays.asList(1,5)).equals(result);
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


<a name="Plus_Minus">
## Plus Minus
</a>

Given an array of integers, calculate which fraction of its elements are positive, which fraction of its elements are negative, and which fraction of its elements are zeroes, respectively.

**Output**

* A decimal representing of the fraction of positive numbers in the array compared to its size.
* A decimal representing of the fraction of negative numbers in the array compared to its size.
* A decimal representing of the fraction of zeroes in the array compared to its size.


**Sample Input**

```bash
[-4,3,-9,0,4,1]
```


**Sample Output**

```bash
0.5
0.333333
0.166667
```

**Explanation**

There are `3` positive numbers, `2` negative numbers, and `1` zero in the array.

The respective fractions of positive numbers, negative numbers and zeroes are: `$\frac{3}{6} = 0.5, \frac{2}{6} = 0.333333, \frac{1}{6} = 0.166667$` respectively.

**Solution**

```java
import java.util.List;
import java.util.Arrays;
import java.util.ArrayList;

public class PlusMinusFinder {

  private float[] find(List<Integer> numbers){
    float[] result = new float[3];
    result[0] = numbers.stream().filter(it -> it > 0).count() / (float)numbers.size();
    result[1] = numbers.stream().filter(it -> it < 0).count() / (float)numbers.size();
    result[2] = numbers.stream().filter(it -> it == 0).count() / (float)numbers.size();
    return result;
  }

  public static void main(String[] args){
    List<Integer> numbers = Arrays.asList(-4,3,-9,0,4,1);
    float[] result = new PlusMinusFinder().find(numbers);
    assert 0.5f == result[0];
    assert 0.33333334f == result[1];
    assert 0.16666667f == result[2];
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

Colleen is having a birthday! She will have a cake with one candle for each year of her age. When she blows out the candles, she’ll only be able to blow out the tallest ones.

**Output**

A integer denoting the number of candles the can be blown out.

**Sample Input**

```bash
[3,2,1,3]
```

**Sample Output**

```bash
2
```

**Explanation**

The maximum candle height is 3 and there are two candles of that height.

**Solution**

```java
import java.util.List;
import java.util.Arrays;
import java.util.Optional;

public class BirthdayCakeCandlesCounter {

  private Integer count(List<Integer> sizes){
    Optional<Integer> max = sizes.stream().max(Integer::compare);
    Long result = sizes.stream().filter(it -> it == max.get()).count();
    return result.intValue();
  }

  public static void main(String[] args){
    List<Integer> sizes = Arrays.asList(3, 2, 1, 3);
    Integer result = new BirthdayCakeCandlesCounter().count(sizes);
    assert 2 == result;
  }

}
```


<a name="Breaking_Records">
## Breaking The Records
</a>

Maria plays `n` games of college basketball in a season. Because she wants to go pro, she tracks her points scored per game sequentially in an array defined as:

```bash
score = [s0, s1, s2, ..., sn]
```

After each game `i`, she checks to see if score `s` breaks her record for most or least points scored so far during that season.

Given Maria's array of `scores` for a season of `n` games, find the number of times she breaks her record for most and least points scored during the season.

**Note:** Assume her records for most and least points at the start of the season are the number of points scored during the first game of the season.


**Sample Input**

```bash
[10, 5, 20, 20, 4, 5, 2, 25, 1]
```

**Sample Output**

```bash
[2, 4]
```

**Explanation**

She broke her best record twice and her worst record four times.


**Solution**

```java
import java.util.List;
import java.util.Arrays;

public class BreackingRecordsCounter {

  private Pair count(List<Integer> scores){
    Integer highest = scores.get(0);
    Integer lowest = scores.get(0);
    Integer highestCounter = 0;
    Integer lowestCoutner = 0;

    for(Integer score : scores){
      if(score > highest) {
        highestCounter++;
        highest = score;
      }
      if(score < lowest){
        lowestCoutner++;
        lowest = score;
      }
    }

    return new Pair(highestCounter, lowestCoutner);
  }

  public static void main(String[] args){
    List<Integer> scores = Arrays.asList(10, 5, 20, 20, 4, 5, 2, 25, 1);
    Pair result = new BreackingRecordsCounter().count(scores);
    assert 2 == result.getHighest();
    assert 4 == result.getLowest();
  }

}
```


Pair is just a POJO for containing highest and lowest values.

This solution kind is `O(N)` since the algorithm performance will grow linearly and in direct proportion to the size of the input data set.

<a name="Birthday_Chocolate">
## Birthday Chocolate
</a>

Lily has a chocolate bar consisting of a row of `n` squares where each square has an integer written on it. She wants to share it with Ron for his birthday, which falls on month `m` and day `d`. Lily wants to give Ron a piece of chocolate only if it contains `m` consecutive squares whose integers sum to `d`.

Given `m`, `d`, and the sequence of integers written on each square of Lily's chocolate bar, how many different ways can Lily break off a piece of chocolate to give to Ron?

For example, if `m=2`, `d=3` and the chocolate bar contains `n` squares with the integers

```bash
[1, 2, 1, 3, 2]
```

written on them from left to right, the following diagram shows two ways to break off a piece:

<img src="/img/techtalks/algorithms/birthday_chocolate.png" />

**Input Format**

* Integer array describing the respective values of `s0, s1, s2, s3, ..., sn` (the numbers written on each consecutive square of chocolate).
* Integer Ron's birth day
* Integer Ron's birth month

**Constraints**

* `$1 \le n \le 100$`
* `$1 \le s1 \le 5, where(0 \le i \le n)$`
* `$1 \le d \le 31$`
* `$1 \le m \le 12$`

**Output Format**

Print an integer denoting the total number of ways that Lily can give a piece of chocolate to Ron.

**Sample Input**

```bash
numbers = [1, 2, 1, 3, 2]
day = 3
month = 2
```

**Sample Output**

```bash
2
```

**Solution**

```java
import java.util.Map;
import java.util.List;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class BirthdayChocolateCalculator {

  private Integer compute(List<Integer> numbers, final Integer month, final Integer day) {
    Integer types = 0;

    List<List<Integer>> sets =
      IntStream.range(1, numbers.size() - month)
      .mapToObj(it -> numbers.subList(it - 1, it + month - 1))
      .collect(Collectors.toList());

    for(List<Integer> set : sets){
      if(set.stream().mapToInt(Integer::intValue).sum() == day){
        types++;
      }
    }

    return types;
  }

  public static void main(String[] args){
    Integer month = 2;
    Integer day = 3;
    List<Integer> numbers = Arrays.asList(1, 2, 1, 3, 2);
    Integer result = new BirthdayChocolateCalculator().compute(numbers, month, day);
    assert 2 == result;
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

<a name="Bon_Appetit">
## Bon Appetit
</a>

Anna and Brian order `$N$` items at a restaurant, but Anna declines to eat any of the `$Kth$` item (where `$0 \le K \le N$`) due to an allergy. When the check comes, they decide to split the cost of all the items they shared; however, Brian may have forgotten that they didn't split the `$Kth$` item and accidentally charged Anna for it.

You are given `$N, K,$` the cost of each of the  items, and the total amount of money that Brian charged Anna for her portion of the bill. If the bill is fairly split, print Bon Appetit; otherwise, print the amount of money that Brian must refund to Anna. It is guaranteed that the amount will always be an integer.

**Input Format**

An integer denoting `$K$`(the `$0$`-based index of the item that Anna did not eat). 
An integer arrray where each integer `$i$` denotes the cost, `$c[i]$`, of item  `$i$` (where `$0 \le i \le N$`).

**Constraints**

* `$0 \le K \le n$`
* `$0 \le c[i] \le 10^4$` 

**Output Format**

The difference (i.e., `$Bcharged - Bactual  $`) that Brian must refund to Anna (it is guaranteed that this will always be an integer).

**Sample Input**

```bash
itemAvoidided = 1
prices = [3, 10, 2, 9]
```

**Sample Output**

```bash
5
```

**Explanation**

Anna didn't eat item `$c[i] = 10  $`, but she shared the rest of the items with Brian. The total cost of the shared items is  `$3 + 2 + 9 = 14$`and, split in half, the cost per person is `$Bactual = 7$`. Brian charged her `$Bcharged = 12$` for her portion of the bill, which is more than the `$7$` dollars worth of food that she actually shared with him. Thus, we get the amount Anna was overcharged, `$Bcharged-Bactual = 12 - 7 = 5$`.

**Solution**

```java
import java.util.List;
import java.util.Arrays;
import java.util.ArrayList;
import java.util.LinkedList;

public class BonAppetitCalculator {

  private Integer compute(List<Integer> prices, int itemAvoided){
    Integer charged = prices.stream().mapToInt(Integer::intValue).sum() / 2;
    prices.remove(itemAvoided);
    Integer fair = prices.stream().mapToInt(Integer::intValue).sum() / 2;
    return charged - fair;
  }

  public static void main(String[] args){
    List<Integer> prices = new LinkedList<Integer>(Arrays.asList(3, 10, 2, 9));
    int itemAvoided = 1;
    Integer result = new BonAppetitCalculator().compute(prices, itemAvoided);
    assert 5 == result;
  }
  
}
```

To download the code:

```bash
git clone https://github.com/josdem/algorithms-workshop.git
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
