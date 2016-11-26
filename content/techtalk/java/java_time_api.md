+++
categories = ["techtalk", "code"]
date = "2016-11-26T10:46:26-06:00"
tags = ["josdem", "techtalks", "programming", "technology", "Java"]
title = "Java Time API"

+++

Some important things that we have to know about the `java.time API` is the following:

* Is an evolution from the previous `java.util.Date (Calendar, TimeZone & DateFormat)`
* Instances of time/date now are immutable. (This is importat for lambda expressions)
* Time and Date operations now are thread safe.
* The API support strong typing, which enables you to write better code.

*A quick glance to the Date methods*

```java
import java.time.LocalDate;
import java.time.Month;

public class DateOperations {

  private void operations(){
    LocalDate today = LocalDate.now();
    System.out.println("today in YYYY-MM-DD format: " + today);
    System.out.println("Day of the week: " + today.getDayOfWeek());
    System.out.println("Next week: " + today.plusDays(7));
    System.out.println("Is a Leap Year?: " + today.isLeapYear());

    LocalDate christmas = LocalDate.of(2016, Month.DECEMBER, 25);
    System.out.println("Today is before Christmas?: " + today.isBefore(christmas));
  }

  public static void main(String[] args){
    new DateOperations().operations();
  }
}
```

*output*

```bash
today in YYYY-MM-DD format: 2016-11-26
Day of the week: SATURDAY
Next week: 2016-12-03
Is a Leap Year?: true
Today is before Christmas?: true
```

**Calculating Between Days**

When you want to perform calculations between days, temporal unit interface unit can be used, since represents a unit of time. Enumeration ChronoUnit actually implements our temporal unit interface. By importing `java.time.temporal,ChronoUnit.*` you can use enumerations such as days and you can use methods as between.

*example*

```java
import java.time.Month;
import java.time.Period;
import java.time.LocalDate;
import static java.time.temporal.ChronoUnit.*;

public class BetweenDays {

  private void compute(){
    LocalDate christmas = LocalDate.of(2016, Month.DECEMBER, 25);
    LocalDate today = LocalDate.now();
    long days = DAYS.between(today, christmas);
    System.out.println("There are " + days + " shopping days until Christmas");
  }

  public static void main(String[] args){
    new BetweenDays().compute();
  }
}
```

*output*

```bash
There are 29 shopping days until Christmas
```

A period class also have a between method and you can use to figure out how many days or months are between dates.

*example*

```java
import java.time.Month;
import java.time.Period;
import java.time.LocalDate;
import static java.time.temporal.ChronoUnit.*;

public class BetweenDays {

  private void compute(){
    LocalDate christmas = LocalDate.of(2016, Month.DECEMBER, 25);
    LocalDate today = LocalDate.now();
    long days = DAYS.between(today, christmas);
    System.out.println("There are " + days + " shopping days until Christmas");

    Period untilChristmas = Period.between(today, christmas);
    System.out.println("There are " + untilChristmas.getMonths() + " months and " + untilChristmas.getDays() + " days until Christmas");
  }

  public static void main(String[] args){
    new BetweenDays().compute();
  }
}
```

*output*

```bash
There are 29 shopping days until Chrismas
There are 0 months and 29 days until Christmas
```

[Return to the main article](/techtalk/java)
