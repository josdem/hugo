+++
categories = ["techtalk", "code","Java"]
date = "2016-06-23T15:24:28-05:00"
tags = ["josdem", "techtalks", "programming", "technology","Java"]
title = "FizzBuzz"

+++

Fizz buzz is a group word game for children to teach them about division. Players take turns to count incrementally, replacing any number divisible by three with the word “fizz”, and any number divisible by five with the word “buzz”.

Our goal is to test a starting round of fizz buzz from 1 to 100 as follow:

1, 2, Fizz, 4, Buzz, Fizz, 7, 8, Fizz, Buzz...

**Java legacy**

```java
public class FizzBuzz {

  FizzBuzz(){
    for(int i=0; i<100; i++){
      if(i%3==0 && i%5==0){
        System.out.println("FizzBuzz");
      } else if(i%3==0){
        System.out.println("Fizz");
      } else if(i%5==0){
        System.out.println("Buzz");
      } else {
        System.out.println(i);
      }
    }
  }

  public static void main(String[] args){
    new FizzBuzz();
  }

}
```

That was a really brute force solution.

**Java 8**

```java
import java.util.stream.IntStream;

public class FizzBuzz {

  FizzBuzz(){
    IntStream.range(0, 100).forEach( i ->
        System.out.println(fizzBuzzify(i))
    );
  }

  private String fizzBuzzify(final int value) {
    StringBuffer result = new StringBuffer();

    if (value % 3 == 0) {
      result.append("Fizz");
    }
    if (value % 5 == 0) {
      result.append("Buzz");
    }
    return result.length() > 0 ? result.toString() : Integer.toString(value);
  }

  public static void main(String[] args){
    new FizzBuzz();
  }

}
```

[Return to the main article](/techtalk/java)
