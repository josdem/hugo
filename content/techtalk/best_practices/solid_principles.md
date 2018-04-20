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

**Open-closed Principle**

Objects or entities should be open for extension, but closed for modification.

*Open for extension means that we should be able to add new features or components to the application without breaking existing code. Closed for modification means that we should not introduce breaking changes to existing functionality, because that would force you to refactor a lot of existing code ~ Eric Elliott*

Let's consider the following employee salary monlty calculator:

```java
package com.jos.dem.solid.ocp;

import java.math.BigDecimal;

import com.jos.dem.solid.ocp.EmployeeType;

public class Employee {

	private final BigDecimal monltySalary = new BigDecimal(100.00);
	private final BigDecimal bonus = new BigDecimal(20.00);
	private final BigDecimal commission = new BigDecimal(10.00);

	public BigDecimal getPaymentAmount(EmployeeType type) {
		
		switch(type){
			case ENGINEER:
				return monltySalary;
			case SALESMAN:
				return monltySalary.add(commission);
			case MANAGER:
				return monltySalary.add(bonus);	
			default:
				throw new RuntimeException("Incorrect Employee");
		}
	}

}
```

Here is the Junit test to cover this functionality:

```java
package com.jos.dem.solid.ocp;

import static org.junit.Assert.assertEquals;
import java.math.BigDecimal;
import org.junit.Test;
import com.jos.dem.solid.ocp.EmployeeType;

public class EmployeeTest {
	
	private Employee employee = new Employee();
	
	@Test
	public void shouldGetEngineerSalary() {
		BigDecimal salary = new BigDecimal(100.00);
		assertEquals(salary, employee.getPaymentAmount(EmployeeType.ENGINEER));
	}
	
	@Test
	public void shouldGetSalesmanSalary() {
		BigDecimal salary = new BigDecimal(110.00);
		assertEquals(salary, employee.getPaymentAmount(EmployeeType.SALESMAN));
	}
	
	@Test
	public void shouldGetManagerSalary() {
		BigDecimal salary = new BigDecimal(120.00);
		assertEquals(salary, employee.getPaymentAmount(EmployeeType.MANAGER));
	}

}
```

If we need to create a new employee type, we need to add a new case in the switch conditional, so let's create a employee abstraction instead.

Employee:

```java
package com.jos.dem.solid.ocp;

import java.math.BigDecimal;

public interface Employee {
	
	final BigDecimal monthlySalary = new BigDecimal(100);
	BigDecimal getPaymentAmount();

}
```

Engineer:

```java
package com.jos.dem.solid.ocp;

import java.math.BigDecimal;

public class Engineer implements Employee {
	
	public BigDecimal getPaymentAmount() {
		return monthlySalary;
	}

}
```

Manager:

```java
package com.jos.dem.solid.ocp;

import java.math.BigDecimal;

public class Manager implements Employee {
	
	private BigDecimal bonus = new BigDecimal(20);

	public BigDecimal getPaymentAmount() {
		return monthlySalary.add(bonus);
	}

}
```

Salesman:

```java
package com.jos.dem.solid.ocp;

import java.math.BigDecimal;

public class Salesman implements Employee {
	
	private BigDecimal commission = new BigDecimal(10);

	public BigDecimal getPaymentAmount() {
		return monthlySalary.add(commission);
	}

}
```

Here is our `EmployeeTest` modified:

```java
package com.jos.dem.solid.ocp;

import static org.junit.Assert.assertEquals;
import java.math.BigDecimal;
import org.junit.Test;

public class EmployeeTest {
	
	@Test
	public void shouldGetEngineerSalary() {
		BigDecimal salary = new BigDecimal(100);
		Employee engineer = new Engineer();
		assertEquals(salary, engineer.getPaymentAmount());
	}
	
	@Test
	public void shouldGetSalesmanSalary() {
		BigDecimal salary = new BigDecimal(110);
		Employee salesman = new Salesman();
		assertEquals(salary, salesman.getPaymentAmount());
	}
	
	@Test
	public void shouldGetManagerSalary() {
		BigDecimal salary = new BigDecimal(120);
		Employee manager = new Manager();
		assertEquals(salary, manager.getPaymentAmount());
	}

}
```

That's it, if we need to add a new employee type, we only need to create a new employee implementation.

**Liskov substitution principle**

Every subclass/derived class should be substitutable for their base/parent class.

In other words, a subclass should override the parent class methods in a way that does not break functionality from a client’s point of view.

```java
package com.jos.dem.solid.lsp;

import static org.junit.Assert.assertEquals;

import java.util.List;
import java.util.Arrays;
import java.math.BigDecimal;
import org.junit.Test;

public class EmployeeTest {
	
	@Test
  public void shouldGetTotalSalary() {
    BigDecimal expectedTotal = new BigDecimal(330);
    Employee engineer = new Engineer();
    Employee manager = new Manager();
    Employee salesman = new Salesman();

    List<Employee> employees = Arrays.asList(engineer, manager, salesman);

    assertEquals(expectedTotal, employees.stream()
      .map(Employee::getPaymentAmount)
      .reduce(BigDecimal.ZERO, BigDecimal::add));
	}

}
```

In this case we are iterating over employees concrete implementations and getting payment amount using their abstraction, that's it, from client's point of view we can use a concrete or abstraction elements.

**Interface segregation principle**

Clients should not be forced to implement unnecessary methods which they will not use.

Let's consider the following `Employee` interface:

```java
package com.jos.dem.solid.isp;

import java.math.BigDecimal;

public interface Employee {
  BigDecimal getBaseAmount();
}
```

And this two interfaces that extends `Employee` interface:

```java
package com.jos.dem.solid.isp;

import java.math.BigDecimal;

public interface Partner extends Employee {
  BigDecimal getProfits();
}
```

Contractor:

```java
package com.jos.dem.solid.isp;

import java.math.BigDecimal;

public interface Contractor extends Employee {
  BigDecimal getBonus();
}
```

In this case a Partner is a special kind of worker who has a base payment and a profit payment, a contractor is another kind of worker who has base payment and a bonus. Here is the concrete implementations from those interfaces.

Partner:

```java
package com.jos.dem.solid.isp;

import java.math.BigDecimal;

public class PartnerImpl implements Partner {

  private static final BigDecimal BASE_SALARY = new BigDecimal(100);
  private static final BigDecimal PROFIT_PERCENTAGE = new BigDecimal(20);
  private Integer hours;

  public PartnerImpl(Integer hours){
    this.hours = hours;
  }

	@Override
	public BigDecimal getBaseAmount() {
		return BASE_SALARY.multiply(new BigDecimal(hours));
	}

	@Override
	public BigDecimal getProfits() {
		return getBaseAmount().multiply(PROFIT_PERCENTAGE).divide(new BigDecimal(100));
  }

}
```

Contractor:

```java
package com.jos.dem.solid.isp;

import java.math.BigDecimal;

public  class ContractorImpl implements Contractor {

  private static final BigDecimal BASE_SALARY = new BigDecimal(80);
  private static final BigDecimal BASE_BONUS = new BigDecimal(10);
  private Integer hours;

  public ContractorImpl(Integer hours){
    this.hours = hours;
  }

	@Override
	public BigDecimal getBaseAmount() {
		return BASE_SALARY.multiply(new BigDecimal(hours));
	}

	@Override
	public BigDecimal getBonus() {
		return BASE_BONUS.multiply(new BigDecimal(hours));
  }

}
```

This strategy represent how we can create several interfaces so our implementation decide which interfaces implement in order to avoid to implement unnecessary methods.

Employee Test

```java
package com.jos.dem.solid.isp;

import static org.junit.Assert.assertEquals;

import java.math.BigDecimal;
import java.util.Arrays;
import java.util.List;

import org.junit.Test;

public class EmployeeTest {

  @Test
  public void shouldGetTotalPartnersAmount(){
    BigDecimal expectedTotal = new BigDecimal(5400);    
    List<Partner> partners = Arrays.asList(new PartnerImpl(10), new PartnerImpl(15), new PartnerImpl(20));

    assertEquals(expectedTotal, partners.stream()
      .map(it -> it.getBaseAmount().add(it.getProfits()))
      .reduce(BigDecimal.ZERO, BigDecimal::add));
  }

  @Test
  public void shouldGetTotalContractorAmount(){
    BigDecimal expectedTotal = new BigDecimal(4050);    
    List<Contractor> contractors = Arrays.asList(new ContractorImpl(10), new ContractorImpl(15), new ContractorImpl(20));

    assertEquals(expectedTotal, contractors.stream()
      .map(it -> it.getBaseAmount().add(it.getBonus()))
      .reduce(BigDecimal.ZERO, BigDecimal::add));
  }

}
```

**Dependency inversion principle**

Entities must depend on abstractions not on concretions.

The classical use of this principle of `BeanFactory` in spring framework. In spring framework, all modules are provided as separate components which can work together by simply injected dependencies in other module.

Let's consider a `PersonService` and inject a `PersonRepository` as dependency.

```java
package com.jos.dem.solid.dip;

import java.util.List;

public class PersonService {

  private PersonRepository personRepository;

  public void setPersonRepository(PersonRepository personRepository){
    this.personRepository = personRepository;
  }

  public List<Person> getAll(){
    return personRepository.findAll();
  }

}
```

Person Repository

```java
package com.jos.dem.solid.dip;

import java.util.List;

public interface PersonRepository{
  List<Person> findAll();
}
```

That's it, we are injecting person repository but it is an abstraction, person service depends on abstraction not concretions. This is the implementation:

```java
package com.jos.dem.solid.dip;

import java.util.Arrays;
import java.util.List;

public class PersonMemoryRepository implements PersonRepository {

  private List<Person> persons = Arrays.asList(new Person("josdem"));

	@Override
	public List<Person> findAll() {
		return persons;
	}

}
```

In that case, we are using an in-memory repository and we can easily change it to a database repository and our service will not notice that change.

```java
package com.jos.dem.solid.dip;

public class Person {

  private String nickname;

  public Person(String nickname){
    this.nickname = nickname;
  }

  public String getNickname(){
    return this.nickname;
  }

}
```

To download the project:

```bash
git clone https://github.com/josdem/solid-principles.git
```

To run the project:

```bash
gradle test
```

[Return to the main article](/techtalk/best_practices)
