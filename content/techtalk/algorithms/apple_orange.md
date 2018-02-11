+++
date = "2018-02-11T20:52:26-05:00"
tags = ["josdem", "techtalks","programming","technology","algorithms", "coding challenge","code"]
categories = ["techtalk", "code"]
title = "Apple and Orange"
+++

Sam's house has an apple tree and an orange tree that yield an abundance of fruit. Sam’s two children, Larry and Rob, decide to play a game in which they each climb a tree and throw fruit at their (Sam’s) house. Each fruit that lands on the house scores one point for the one who threw it. Larry climbs the tree on the left (the apple), and Rob climbs the one on the right (the orange).

<img src="/img/techtalks/algorithms/apple_orange.png"/>

Larry climbs the apple tree at point , and Rob climbs the orange tree at point . Sam’s house stands between points  and . Values increase from left to right.

You will be given a list of distances the fruits are thrown. Negative distances indicate travel left and positive distances, travel right. Your task will be to calculate the scores for Larry and Rob and report them each on a separate line.

**Constraints**

* `$1 \le s,t,a,b \le 10^5$`
* `$-10^5 \le d \le 10^5$`
* `$a \le s \le t \le b$`

**Input Format**

* Pair numbers containing integers denoting the respective values of `s` and `t`.
* Pair numbers containing integers denoting the respective values of `a` and `b`.
* Integers array denoting the respective distances that each apple falls from point `a`.
* Integers array denoting the respective distances that each orange falls from point `b`.


**Sample Input**

```bash
[7, 11]
[5, 15]
[-2, 2, 1]
[5, -6]
```

**Sample Output**

```bash
[1, 1]
```

**Explanation**

* The first apple falls at position `5 - 2 = 3`
* The second apple falls at position `5 + 2 = 7`
* The third apple falls at position `5 + 1 = 6`
* The first orange falls at position `15 + 5 = 7`
* The second orange falls at position `15 - 6 = 9`

Only one fruit (the second apple) falls within the region between `7` and `11`, so Larry’s score is `1`.
Only the second orange falls within the region between `7` and `11`, so Rob’s score is `1`.

**Solution Java 7**

```java
import java.util.List;
import java.util.ArrayList;
import java.util.Arrays;

public class AppleOrangeScorer {

  private Pair score(
      Pair houseWidth,
      Pair treeDistance,
      List<Integer>larryThrows,
      List<Integer>robThrows){

    Pair result = new Pair();

    Integer larryScore = 0;
    Integer robScore = 0;

     for(Integer lthrow: larryThrows){
       if(treeDistance.getFirst() + lthrow >= houseWidth.getFirst() &&
          treeDistance.getFirst() + lthrow <= houseWidth.getLast()){
         larryScore++;
       }
     }

    for(Integer rthrow: robThrows){
       if(treeDistance.getLast() + rthrow >= houseWidth.getFirst() &&
          treeDistance.getLast() + rthrow <= houseWidth.getLast()){
         robScore++;
       }
     }

    result.setFirst(larryScore);
    result.setLast(robScore);

    return result;
  }

  public static void main(String[] args){
    Pair houseWidth = new Pair(7, 11);
    Pair treeDistance = new Pair(5, 15);

    List<Integer> larryThrows = Arrays.asList(-2, 2, 1);
    List<Integer> robThrows = Arrays.asList(5, -6);

    Pair result = new AppleOrangeScorer().score(houseWidth, treeDistance, larryThrows, robThrows);

    assert 1 == result.getFirst();
    assert 1 == result.getLast();
  }

}

class Pair {
  private Integer first;
  private Integer last;

  public Pair(){}

  public Pair(Integer first, Integer last){
    this.first = first;
    this.last = last;
  }

  public void setFirst(Integer first){
    this.first = first;
  }

  public void setLast(Integer last){
    this.last = last;
  }

  public Integer getFirst(){
    return first;
  }

  public Integer getLast(){
    return last;
  }
}
```

This solution kind is `O(N)` since the algorithm performance will grow linearly and in direct proportion to the size of the input data set.

Let's consider the Java 8 version using lambda

```java
import java.util.List;
import java.util.ArrayList;
import java.util.Arrays;

public class AppleOrangeScorer {

  private Pair score(
      Pair houseWidth,
      Pair treeDistance,
      List<Integer>larryThrows,
      List<Integer>robThrows){

    Pair result = new Pair();

   Long larryScore = larryThrows.stream().filter(it -> treeDistance.getFirst() + it >= houseWidth.getFirst() && treeDistance.getFirst() + it <= houseWidth.getLast()).count();
   Long robScore = robThrows.stream().filter(it -> treeDistance.getLast() + it >= houseWidth.getFirst() && treeDistance.getLast() + it <= houseWidth.getLast()).count();

    result.setFirst(larryScore.intValue());
    result.setLast(robScore.intValue());

    return result;
  }

  public static void main(String[] args){
    Pair houseWidth = new Pair(7, 11);
    Pair treeDistance = new Pair(5, 15);

    List<Integer> larryThrows = Arrays.asList(-2, 2, 1);
    List<Integer> robThrows = Arrays.asList(5, -6);

    Pair result = new AppleOrangeScorer().score(houseWidth, treeDistance, larryThrows, robThrows);

    assert 1 == result.getFirst();
    assert 1 == result.getLast();
  }

}

class Pair {
  private Integer first;
  private Integer last;

  public Pair(){}

  public Pair(Integer first, Integer last){
    this.first = first;
    this.last = last;
  }

  public void setFirst(Integer first){
    this.first = first;
  }

  public void setLast(Integer last){
    this.last = last;
  }

  public Integer getFirst(){
    return first;
  }

  public Integer getLast(){
    return last;
  }
}
```

To Download the code:


```bash
git clone https://github.com/josdem/algorithms-workshop.git
cd apple_orange
```

To run the code:

```bash
javac AppleOrangeScorer.java
java -ea AppleOrangeScorer
```


[Return to the main article](/techtalk/algorithms)


