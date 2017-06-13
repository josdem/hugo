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
import java.util.stream.IntStream;
import java.util.ArrayList;

public class CollectionSum {

  private Integer sum(List<Integer> array){
    return array.stream().mapToInt(it -> it).sum();
  }

  public static void main(String[] args){
    List<Integer> array = new ArrayList<Integer>();
    array.add(1);
    array.add(2);
    array.add(3);
    array.add(4);
    array.add(10);
    array.add(11);
    Integer result = new CollectionSum().sum(array);
    System.out.println(result);
  }

}
```

## Most popular in the array

Assume I have an array that looks like the following:

```groovy
def array = [34,31,34,56,12,35,24,34,69,18]
```

Write a function that can determine what is the most popular in the array, in this case "34" because it is the number that appears the most often.

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

The power set is the set of all subsets of a set. For example, the power set of the set {a, b, c} consists of the sets:

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
