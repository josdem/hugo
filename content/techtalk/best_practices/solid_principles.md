+++
title =  "SOLID Principles"
date = "2018-04-20T12:10:46-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code"]
+++

In object-oriented programming SOLID is a term developed by [Robert C. Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) and the intention is to describe five important design principles to create good software, those concepts are:

* S — Single responsibility principle
* O — Open closed principle
* L — Liskov substitution principle
* I — Interface segregation principle
* D — Dependency Inversion principle

**Single responsibility principle**

Every class should have a single responsibility, and that responsibility should be entirely encapsulated by the class. When a class has more than one reason to be changed, it is more fragile, so a change in one location might lead to some unexpected behavior in totally other places.

Let's consider the following car class:

```java
package com.jos.dem.solid.srp;

public class Car {
  private static final int MAX_FUEL = 40;
  private int fuel;  

  public void fillUp() {
    this.fuel = MAX_FUEL;
  }

  public boolean isFull(){
    return fuel == MAX_FUEL;
  }

  public boolean isEmpty(){
    return fuel == 0;
  }

}
```

And here is the test to cover this functionality:

```groovy
package com.jos.dem.solid.srp

import spock.lang.Specification

class CarSpec extends Specification {

  void "should start with empty tank"(){
    when:'A car'
      Car car = new Car()
    then:'We expect is empty gas'  
      car.isEmpty() == true
  }

  void "should do a gas fill up"(){
    given:'A car'
      Car car = new Car()
    when:'We do a gas fill up'        
      car.fillUp()
    then:'Car is full of gas'  
      car.isFull() == true
  }

}
```

But `fillUp()` gas should NOT be a car's responsibility. Applying Single Responsability principle we need to split that responsability and create another class:

```java
package com.jos.dem.solid.srp;

public class FuelPump {

  public void reFuel(Car car){
    while(!car.isFull()){
      car.increment();
    }
  }

}
```

Here is our car class modified:

```java
package com.jos.dem.solid.srp;

public class Car {
  private static final int MAX_FUEL = 40;
  private int fuel;  

  public void increment() {
    this.fuel++;
  }

  public boolean isFull(){
    return fuel == MAX_FUEL;
  }

  public boolean isEmpty(){
    return fuel == 0;
  }

}
```

Here is the Car's Spock test:

```groovy
package com.jos.dem.solid.srp

import spock.lang.Specification

class CarSpec extends Specification {

  void "should start with empty tank"(){
    when:'A car'
      Car car = new Car()
    then:'We expect is empty gas'  
      car.isEmpty() == true
  }

}
```

And FuelPump Spock test:

```groovy
package com.jos.dem.solid.srp

import spock.lang.Specification

class FuelPumpSpec extends Specification {

  FuelPump fuelPump = new FuelPump()

  void "should fuel a car"(){
    given:'A car'
      Car car = new Car()
    when:'We do a gas fill up'        
      fuelPump.reFuel(car)
    then:'Car is full of gas'  
      car.isFull() == true

  }

}
```


[Return to the main article](/techtalk/best_practices)


