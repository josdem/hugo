+++
title = "Replace Temp With Query"
date = "2015-08-29T18:49:03-05:00"
categories = ["techtalk", "code","refactoring"]
tags = ["josdem", "techtalks", "programming", "technology","clean code", "refactoring"]
description = "You are using temporary variable to hold the result of an expression. Extract the expression into a method. Replace all the references to the temp with the new method. The new method can then be used then in other methods."
+++

You are using temporary variable to hold the result of an expression. Extract the expression into a method. Replace all the references to the temp with the new method. The new method can then be used then in other methods.

The problem with temps is that they are temporary and local. Because they can be seen only in the context of the method in wich they are used, temps tend to encourage longer methods, because that’s the only way you can reach the temp. By replacing the temp with query method, any method in the class can get at the information. That helps a lot in coming up with cleaner code for the class.

Consider the following class:

```java
package com.josdem.refactoring;

public class DiscountCalculator {

   public Double getTotal(Integer quantity, Integer itemPrice) {
    Integer basePrice = quantity * itemPrice;

    Double = discountFactor;
    if (basePrice > 1000) {
      discountFactor * 0.95;
    } else {
      discountFactor * 0.98;
    }
    return basePrice * discountFactor;

}
```

First I replace the right-hand side of the assignment:

```java
Integer basePrice = getBasePrice();
```

Then I replace all references to that temp with the expression.

```java
public Double getTotal() {
  Double = discountFactor;
  if (getBasePrice() > 1000) {
    discountFactor * 0.95;
  } else {
   discountFactor * 0.98;
  }
  return getBasePrice() * discountFactor;
}

private Integer getBasePrice() {
  return quantity * itemPrice;
}
```

With that done I can extract discountFactor in a similar way.

```java
private Integer getDiscountFactor() {
  return (getBasePrice() > 1000) ? 0.95 : 0.98;
}
```

See how it would have been difficult to extract discountFactor if I had not replaced basePrice with a query. The getPrice method ends up as follow.

```java
private Double getTotal() {
  return getBasePrice() * getDiscountFactor();
}
```

*When I’m working on a method, I like to replace temp with query to get rid of any temporary variables that I can remove. If the temp is used for many things, I use split temporary variable first to make the temp easier to replace.* ~ Martin Fowler.

To download the project:

```bash
git clone https://github.com/josdem/refactoring.git
git fetch
git checkout feature/replace-temp-with-query-setup
git checkout feature/replace-temp-with-query-complete
```

To run the project:

```bash
gradle test
```

Read more, returning to the main article. [Refactoring](/techtalk/refactoring)
