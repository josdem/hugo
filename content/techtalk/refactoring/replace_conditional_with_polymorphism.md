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

Use this JUnit test to cover conditional implementation

```java
package com.josdem.refactoring;

import static org.junit.Assert.assertEquals;
import java.math.BigDecimal;
import org.junit.Test;
import com.josdem.bean.EmployeeType;

public class TestEmployee {
  
  private Employee employee = new Employee();
  
  @Test
  public void shouldGetEngineerSalary() throws Exception {
    BigDecimal salary = new BigDecimal(100.00);
    assertEquals(salary, employee.getPaymentAmount(EmployeeType.ENGINEER));
  }
  
  @Test
  public void shouldGetSalesmanSalary() throws Exception {
    BigDecimal salary = new BigDecimal(110.00);
    assertEquals(salary, employee.getPaymentAmount(EmployeeType.SALESMAN));
  }
  
  @Test
  public void shouldGetManagerSalary() throws Exception {
    BigDecimal salary = new BigDecimal(120.00);
    assertEquals(salary, employee.getPaymentAmount(EmployeeType.MANAGER));
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

Use this JUnit test to cover classes implementation

```java
package com.josdem.refactoring;

import static org.junit.Assert.assertEquals;
import java.math.BigDecimal;
import org.junit.Test;

public class TestEmployee {
  
  @Test
  public void shouldGetEngineerSalary() throws Exception {
    BigDecimal salary = new BigDecimal(100);
    Employee engineer = new Engineer();
    assertEquals(salary, engineer.getPaymentAmount());
  }
  
  @Test
  public void shouldGetSalesmanSalary() throws Exception {
    BigDecimal salary = new BigDecimal(110);
    Employee salesman = new Salesman();
    assertEquals(salary, salesman.getPaymentAmount());
  }
  
  @Test
  public void shouldGetManagerSalary() throws Exception {
    BigDecimal salary = new BigDecimal(120);
    Employee manager = new Manager();
    assertEquals(salary, manager.getPaymentAmount());
  }

}
```

To download the project:

```bash
git clone https://github.com/josdem/refactoring.git
git fetch
git checkout feature/replace-conditional-with-polymorphism-setup
git checkout feature/replace-conditional-with-polymorphism-complete
```

To run the project:

```bash
gradle test
```

Read more, returning to the main article. [Refactoring](/techtalk/refactoring)
