+++
categories = ["algorithms", "pseudocode"]
date = "2016-07-04T21:00:30-05:00"
tags = ["josdem", "techtalks", "algorithms", "power set", "palindrome"]
title = "Algorithms"

+++

A Palindrome is a word phrase, or number that reads the same backward or forward.

```groovy
Boolean isPalindrome(String string){
  string.reverse().equals(string)
}

assert isPalindrome('anitalavalatina')
```

## Biggest Number

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

## Sum a collection

Given an array of  integers, find the sum of its elements.

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

## Staircase

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

## Most popular in the array

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


## Find the center point of coordinate 2d array

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

## Power set

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

To download the code:

```bash
git clone https://github.com/josdem/algorithms-workshop.git
cd simple-algorithms
```


[Return to the main article](/techtalk/algorithms)
