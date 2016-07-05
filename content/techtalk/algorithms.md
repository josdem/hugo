+++
date = "2016-05-09T19:09:56-05:00"
draft = true
title = "Algorithms"

+++

* [Simple algorithms](/techtalk/algorithms/simple)
* [Fastman problem](/techtalk/algorithms/algorithm_fastman)
* [Binary Tree](/techtalk/algorithms/algorithm_binary_tree)
* [Stream Merger](/techtalk/algorithms/algorithm_stream_merger)

## Binary Search
Is a search algorithm that finds the position of a target value within a sorted array.

**BinarySearch.java**

```java
public class BinarySearch{

  int logarithmic(Integer[] data, int key) {
    int startIndex = 0;
    int endIndex = data.length - 1;

    while (startIndex < endIndex) {
      int midIndex = (endIndex - startIndex / 2) + startIndex;
      int midValue = data[midIndex];

      if (key > midValue) {
        startIndex = midIndex++;
      } else if (key < midValue) {
        endIndex = midIndex - 1;
      } else {
        return midIndex;
      }
    }
    return -1;
  }

  public static void main(String[] args){
    BinarySearch binarySearch = new BinarySearch();
    Integer[] myData = {1,2,3,4,5,6,7,8,9,10};
    System.out.println("value: " + binarySearch.logarithmic(myData,5));
  }

}
```

## QuickSort

Quicksort (sometimes called partition-exchange sort) is an efficient sorting algorithm, serving as a systematic method for placing the elements of an array in order.

**QuickSort.java**

```java
public class QuickSort<t extends Comparable<Integer>> {

  public Integer[] sort(Integer[] array) {
    return quicksort(array, 0, array.length - 1);
  }

  private Integer[] quicksort(Integer[] array, int lo, int hi) {
    if (hi > lo) {
      int partitionPivotIndex = (int) (Math.random() * (hi - lo) + lo);
      int newPivotIndex = partition(array, lo, hi, partitionPivotIndex);
      quicksort(array, lo, newPivotIndex - 1);
      quicksort(array, newPivotIndex + 1, hi);
    }
    return (Integer[]) array;
  }

  private int partition(Integer[] array, int lo, int hi, int pivotIndex) {
    Integer pivotValue = array[pivotIndex];
    swap(array, pivotIndex, hi);
    int index = lo;
    for (int i = lo; i < hi; i++) {
      if ((array[i]).compareTo(pivotValue) <= 0) {
        swap(array, i, index);
        index++;
      }
    }
    swap(array, hi, index);
    return index;
  }

  private void swap(Integer[] array, int i, int j) {
    Integer temp = array[i];
    array[i] = array[j];
    array[j] = temp;
  }

}
```

**Sorter.java**

```java
public class Sorter{

  public Integer[] sort(Integer[] data) {
    QuickSort<Integer> sorter = new QuickSort<Integer>();
    return sorter.sort(data);
  }

  public static void main(String[] args){
    Integer myData[] = {7,3,5,1,8,4,2,0,6,9};
    Integer[] result = new Sorter().sort(myData);
    for(Integer integer : result){
      System.out.println("number: " + integer);
    }
  }

}

```


[Return to the main article](/techtalk/techtalks)
