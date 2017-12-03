+++
date = "2017-12-03T12:01:18-06:00"
title = "Digital Sum Sort"
tags = ["josdem", "techtalks","programming","technology","Algorithms","Java"]
categories = ["techtalk", "code","Algorithms"]
+++

Sort sum digits from elements in a collection.

**Explanation**

You are given an array of integers. Sort it in such a way that if `A` comes before `B` then the sum of digits of `A` is less than or equal to the sum of digits of `B`.

**Example**

For:

```
collection = [15, 20, 4, 8]
```

The output should be:

```
digitalSumSort(collection) = [2, 4, 6, 8]
```


**Solution**

```java
import java.util.List;
import java.util.Arrays;
import java.util.ArrayList;
import java.util.stream.Collectors;

public class DigitalSumSorter {

  private List<Integer> sort(List<Integer> numbers){
    List<String> numbersAsString = numbers.stream().map(it -> it.toString()).collect(Collectors.toList());
    List<Integer> result = numbersAsString.stream().map(it -> it.chars().map( ch -> Integer.parseInt(Character.toString((char) ch))).sum()).sorted().collect(Collectors.toList());
    return result;
  }

  public static void main(String[] args){
    List<Integer> numbers = Arrays.asList(15, 20, 4, 8);
    List<Integer> result = new DigitalSumSorter().sort(numbers);
    assert Arrays.asList(2,4,6,8).equals(result);
  }

}
```

To download the code:

```bash
git clone https://github.com/josdem/algorithms-workshop.git
cd digital-sum-sort
```

To run the code:

```bash
javac DigitalSumSorter.java
java -ea DigitalSumSorter
```


[Return to the main article](/techtalk/algorithms)
