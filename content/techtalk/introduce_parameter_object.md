+++
date = "2015-08-29T18:55:13-05:00"
draft = true
title = "Introduce Parameter Object"

+++
You have a group of parameters that naturally go together, replace them with an object.

Often you see a particular group of parameters that tend to be passed together. Several methods may use this group, either on one class or in several classes. Such a group of classes is a data clump and can be replaced with an object that carries all of this data. This refactoring is useful because it reduces the size of parameter list, and long parameter lists are hard to understand.

## Argument as Objects
When a function seems to need more than two or three arguments, it is likely that some of those arguments ought to be wrapped in to a class of their own. Consider, for example, the difference between the two following declarations:

```java
Circle makeCircle(double x, double y, double radius);
Circle makeCircle(Point center, double radius);
```

Reducing the number of arguments by creating objects out of them may seem like cheating, but it’s not. When groups of variables are passed together, the way x and y are in the example above, they are likely part of a concept that deserves a name of its own.

## Example
Consider the following class:

```java
package com.josdem.refactoring;
import java.util.List;

public class Account {

  public Double getTotal(Integer min, Integer max, List entries){
    Double result = 0.00;
    for (Entry entry : entries) {
      if (entry.getMeassure() >= min && entry.getMeassure() <= max){
        result += entry.getValue();
      }
    }
    return result;
  }

}
```

Applying a simple refactor here, we can group min and max in their own class as follow:

```java
package com.josdem.refactoring;

public class Range {

  private final Integer min;
  private final Integer max;

  public Range(Integer min, Integer max) {
    this.min = min;
    this.max = max;
  }

  public Integer getMin() {
    return min;
  }

  public Integer getMax() {
    return max;
  }

}
```

I’ve made the range class immutable; that is, all the values for the range are final and set in the constructor, hence are no methods for mofifying the values. This is a wise move to avoid aliasing bugs. Now applying refactoring, the original class turns to this one:

```java
package com.josdem.refactoring;
import java.util.List;

public class Account {

  public Double getTotal(Range range, List entries){
    Double result = 0.00;
    for (Entry entry : entries) {
      if (entry.getMeassure() >= range.getMin() && entry.getMeassure() <= range.getMax()){
        result += entry.getValue();
      }
    }
    return result;
  }

}
```

Read more, returning to the main article. [Refactoring](/techtalk/refactoring)

