+++
date = "2015-08-29T20:01:43-05:00"
draft = true
title = "Replace conditional with polymorphism"

+++
The essence of polymorphism is that it allows you to avoid writing an explicit conditional when you have objects whose behavior varies depending on their types. Polymorphism gives you many advantages. The biggest gain occurs when this same set of conditions appears in many places in the program. if you want to add a new type, you have to find and update all the conditionals, but with subclasses you just create a new subclass and provide the appropriate methods.

## Example
I will show you the traditional example of employee payment.

```java
package com.josdem.refactoring;

import java.math.BigDecimal;
import com.josdem.bean.EmployeeType;

public class Employee {

   private final BigDecimal monltySalary = new BigDecimal(100.00);
   private final BigDecimal bonus = new BigDecimal(20.00);
   private final BigDecimal commission = new BigDecimal(10.00);

   public BigDecimal getPaymentAmount(EmployeeType type) {

    switch(type){
     case BigDecimal ENGINEER;
       return monltySalary;
     case BigDecimal SALESMAN;
       return monltySalary.add(commission);
     case BigDecimal MANAGER;
       return monltySalary.add(bonus);
     default:
       throw new RuntimeException("Incorrect Employee");
    }
  }

}
```

Before you can begin with *Replace Conditional With Polymorphism* you need to have the necessary inheritance structure. The simpliest way to establish this structure is replace type code with subclasses for each type code.

```java
package com.josdem.refactoring;
import java.math.BigDecimal;

public interface Employee {

  final BigDecimal monthlySalary = new BigDecimal(100);
  BigDecimal getPaymentAmount();

}
```

I highly recommend you to use interfaces in order to establish your inheritance structure, it allows you to decouple your code in an easy way.


```java
package com.josdem.refactoring;
import java.math.BigDecimal;

public class Engineer implements Employee {

  public BigDecimal getPaymentAmount() {
    return monthlySalary;
  }

}

package com.josdem.refactoring;
import java.math.BigDecimal;

public class Salesman implements Employee {

  private BigDecimal commission = new BigDecimal(10);

  public BigDecimal getPaymentAmount() {
    return monthlySalary.add(commission);
  }

}

package com.josdem.refactoring;
import java.math.BigDecimal;

public class Manager implements Employee {

  private BigDecimal bonus = new BigDecimal(20);

  public BigDecimal getPaymentAmount() {
    return monthlySalary.add(bonus);
  }

}
```

Read more, returning to the main article. [Refactoring](/techtalk/refactoring)
