+++
date = "2016-06-07T16:31:36-05:00"
draft = true
title = "Java8"

+++

**forEach() in a Map**

Iterate over a collection in Java 8 is a really nice one, which lets you pass a method reference or a lambda to receive (key, value) pairs one by one.

```java
import java.util.Map;
import java.util.HashMap;

class ForEachMap {

  void iterate(Map<String, Integer> items){
    items.forEach((k,v)->{
      System.out.println("Item : " + k + " Count : " + v);
    });
  }

  public static void main(String[] args){
    Map<String, Integer> items = new HashMap<>();
    items.put("A", 10);
    items.put("B", 20);
    items.put("C", 30);

    new ForEachMap().iterate(items);
  }

}
```

**output**

```
Item : A Count : 10
Item : B Count : 20
Item : C Count : 30
```


**forEach() in a List**

Also, you can loop a List with forEach

```java
import java.util.List;
import java.util.ArrayList;

class ForEachList {

  void iterate(List<String> items){
    items.forEach(System.out::println);
  }

  public static void main(String[] args){
    List<String> items = new ArrayList<String>();
    items.add("A");
    items.add("B");
    items.add("C");

    new ForEachList().iterate(items);
  }

}
```

**output**

```
A
B
C
```

**NOTE:** You can get item from collection like this: `items.forEach(item -> System.out.println(item));`

You can iterate a Set in the same way

[Return to the main article](/techtalk/techtalks)
