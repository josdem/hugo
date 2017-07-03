+++
date = "2016-06-07T16:31:36-05:00"
draft = true
title = "Java Topics"

+++

There are three main features in Java 8 that I love, one of them is the lambda expressions is like syntactic sugar for an anonymous class, it will have enormous implications for simplifying development. The another one is the new `java.util.stream` package provide a Stream API to support functional-style operations on streams of elements. The Stream API is integrated into the Collections API, which enables bulk operations on collections, such as sequential or parallel map-reduce transformations. And lastly but not least the `java.time` API that now are compatible with lambda, thread safe and simplified usage.


* [Java time API](/techtalk/java/java_time_api)
* [Lambda Expressions](/techtalk/java/lambda_expressions)
* [FizzBuzz](/techtalk/java/java_fizz_buzz)
* [Hello AWS Lambda](/techtalk/java/hello_aws_lambda)
* [S3 AWS Lambda](/techtalk/java/s3_aws_lambda)
* [Configuration with Apache Commons](/techtalk/java/configuration_apache_commons)
* [Java NIO File Copy](/techtalk/java/java_nio_copy)
* [CSV with Apache Commons](/techtalk/java/csv_apache_commons)
* [Read Excel with Apache POI](/techtalk/java/apache_poi_excel)
* [Read Filtered Excel with Apache POI](/techtalk/java/apache_poi_excel_filter_reader)
* [GMail Mailbox Reader with POP3](/techtalk/java/mailbox_reader_pop3)
* [GMail Mailbox Reader with IMAP](/techtalk/java/mailbox_reader_imap)
* [Outlook Mailbox Reader with Exchange](/techtalk/java/mailbox_reader_exchange)
* [Streams](/techtalk/java/streams)
* [Stream Filters](/techtalk/java/stream_filters)
* [Stream Collectors](/techtalk/java/stream_collectors)
* [GitHub Repository](https://github.com/josdem/java-topics)


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
