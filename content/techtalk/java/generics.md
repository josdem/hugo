+++
description = "Generics"
date = "2018-04-25T09:57:11-05:00"
tags = ["josdem", "techtalks","programming","technology","generics","java"]
categories = ["techtalk", "code","java"]
title =  "Generics is a great way to prevent bugs at compilation time since provide compile-time type checking and removing risk of ClassCastException that was common while working with collection classes. Another important advantage in using generics is that you can write code that use abstractions instead using concrete classes, so you can create maintainable and extendable code."
+++

Generics is a great way to prevent bugs at compilation time since provide compile-time type checking and removing risk of `ClassCastException` that was common while working with collection classes. Another important advantage in using generics is that you can write code that use abstractions instead using concrete classes, so you can create maintainable and extendable code.

Following example illustrates how we can print an array containing different types using a single Generic method:

```java
import java.util.List;
import java.util.Arrays;

public class ListPrinter {

  private <T> void printList(List<T> collection){
    for(T item: collection){
      System.out.printf("%s ", item);
    }
    System.out.println();
  }

  public static void main(String[] args){
    List<Integer> integers = Arrays.asList(1,2,3,4,5);
    List<Double> doubles = Arrays.asList(1.1,2.2,3.3,4.4,5.5);
    List<Character> string = Arrays.asList('J','O','S','D','E','M');
    ListPrinter printer = new ListPrinter();
    System.out.println("Printing integers");
    printer.printList(integers);
    System.out.println("Printing doubles");
    printer.printList(doubles);
    System.out.println("Printing string");
    printer.printList(string);
  }
}
```

Output:

```bash
Printing integers
1 2 3 4 5
Printing doubles
1.1 2.2 3.3 4.4 5.5
Printing string
J O S D E M
```

Now let's create a generic type that can receive different object types. Imagine you can store String and Integer types in the same generic object.

```groovy
package com.jos.dem.generics

import spock.lang.Specification

class GenericsTypeSpec extends Specification {

  void "should create a generic type and cast to string"(){
    given:'A generic type'
      GenericType<String> type = new GenericType<String>()
    when:'I create a string type'
      type.set('josdem');  
    then:'I expect to get the string'
      'josdem' == type.get();  
  }

  void "should support string and integer types"(){
    given:'A generic type'
      GenericType type = new GenericType()
    when:'I create a string and integer type'
      type.set('josdem');
      type.set(7);
    then:'I expect to get integer'
      7 == type.get();
  }

}
```

GenericType implementation:

```java
package com.jos.dem.generics;

public class GenericType<T> {

  private T type;

  public T get(){
    return this.type;
  }

  public void set(T type){
    this.type = type;
  }

} 
```

**Generic Type Naming Convention**

Java Generic Type Naming convention helps us understanding code easily and having a naming convention is one of the best practices of java programming language. Type parameter names are uppercase letters to make it easily distinguishable from java variables. The most commonly used type parameter names are:

* E – Element (used extensively by the Java Collections, for example List, Set etc.)
* K – Key (Used in Map)
* N – Number
* T – Type
* V – Value (Used in Map)
* S, U. – 2nd, 3rd types

**Bounded Type Parameters**

What you should do when you want to restrict the types that can be used as type arguments in a parameterized type. Let's consider the following example:

```java
package com.jos.dem.generics;

public class NaturalNumber<N extends Number> {
  private N number;

  public NaturalNumber(N number){
    this.number = number;
    if(number.intValue() < 0 ){
      throw new IllegalArgumentException();
    }
  }

  public boolean isEven() {
	  return this.number.intValue() % 2 == 0;
  }

}
```

NaturalNumber test case:

```java
package com.jos.dem.generics;

import static org.junit.Assert.assertFalse;
import static org.junit.jupiter.api.Assertions.assertTrue;

import org.junit.Test;

public class NaturalNumberTest {

  @Test(expected=IllegalArgumentException.class)
  public void shouldNotAcceptNegativeNumbers(){
    new NaturalNumber(-1);
  }

  @Test
  public void shouldKnowIfIsEven(){
    NaturalNumber<Number> naturalNumber = new NaturalNumber<Number>(2);
    assertTrue(naturalNumber.isEven());
  }

  @Test
  public void shouldKnowIfIsNotEven(){
    NaturalNumber<Integer> naturalNumber = new NaturalNumber<Integer>(1);
    assertFalse(naturalNumber.isEven());
  }
  
}
```
As you can see, bounded type parameters allow you to invoke methods defined in the bounds, in this case Number type or subtype, also since you extends Number you can use some defined methods such as `isEven()` and `intValue()`.

This is another example using Generics in a class level and in a method level:

```java
package com.jos.dem.generics;

import static org.junit.Assert.assertEquals;

import org.junit.Test;

public class BoxTest {

  @Test
  public void shouldCreateAnStringBox(){
    String nickname = "josdem";    
    Box<String> box = new Box<String>();
    box.set(nickname);
    assertEquals("java.lang.String", box.getClassTypeName());
  }

  @Test
  public void shouldCreateAnIntegerInspection(){    
    Integer integer = new Integer(10);
    Box<Integer> box = new Box<Integer>();   
    assertEquals("java.lang.Integer", box.getClassNumberName(integer));
  }

}
```

Box implementation:

```java
package com.jos.dem.generics;

public class Box<T> {
  private T type;

  public T get(){
    return this.type;
  }

  public void set(T type){
    this.type = type;
  }

  public String getClassTypeName(){
    return type.getClass().getName();
  }

  public <U extends Number> String getClassNumberName(U unit){
    return unit.getClass().getName();
  }

}
```

**Java Generics Upper Bounded Wildcard**

Upper bounded wildcards are used to relax the restriction on the type of variable in a method. Suppose we want to write a method that will return the sum of numbers in a list.

```java
package com.jos.dem.generics;

import java.util.List;

public class CollectionAdder {

  public Double sum(List<? extends Number> numbers){
    double result = 0;
    for(Number number : numbers){
      result += number.doubleValue();
    }
    return result;
  }
    
 }
```

In this way, we can sum list of Integers or Doubles even though we know that `List<Integer>` and `List<Double>` are not related, this is when upper bounded wildcard is helpful.

Here is the code to test this functionality:

```java
package com.jos.dem.generics;

import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.Arrays;
import java.util.List;

import org.junit.Test;

public class CollectionAdderTest {

  @Test
  public void shouldSumIntegers(){
    CollectionAdder collectionAdder = new CollectionAdder();
    List<Integer> numbers = Arrays.asList(new Integer(10), new Integer(11), new Integer(12));
    Double result = collectionAdder.sum(numbers);
    assertEquals(new Double(33), result);
  }

  @Test
  public void shouldSumDoubles(){
    CollectionAdder collectionAdder = new CollectionAdder();
    List<Double> numbers = Arrays.asList(new Double(10), new Double(11), new Double(12));
    Double result = collectionAdder.sum(numbers);
    assertEquals(new Double(33), result);
  }

}
```
To download the code:

```bash
git clone https://github.com/josdem/java-workshop.git
cd generics
```

To run the code:

```bash
javac ${JAVA_PROGRAM}.java
java -ea ${JAVA_PROGRAM}
```

To test the code:

```bash
gradle test
```

[Return to the main article](/techtalk/java)
