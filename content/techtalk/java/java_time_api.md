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

public class DateTimeOperations {

  private void operations(){
    LocalDate today = LocalDate.now();
    System.out.println("today in YYYY-MM-DD format: " + today);
    System.out.println("Day of the week: " + today.getDayOfWeek());
    System.out.println("Next week: " + today.plusDays(7));
    System.out.println("Is a Leap Year?: " + today.isLeapYear());

    LocalDate chrismas = LocalDate.of(2016, Month.DECEMBER, 25);
    System.out.println("Today is bafore Chirsmas?: " + today.isBefore(chrismas));
  }

  public static void main(String[] args){
    new DateTimeOperations().operations();
  }
}
```

*output*

```bash
today in YYYY-MM-DD format: 2016-11-26
Day of the week: SATURDAY
Next week: 2016-12-03
Is a Leap Year?: true
Today is bafore Chirsmas?: true
```

**Calculating Between Days**

When you want to perform calculations between days, temporal unit interface unit can be used, since represents a unit of time. Enumeration ChronoUnit actually implements our temporal unit interface. By importing `java.time.temporal,ChronoUnit.*` you can use enumerations such as days and you can use methods as between.

*example*

```java
import java.time.Month;
import java.time.LocalDate;
import static java.time.temporal.ChronoUnit.*;

public class BetweenDays {

  private void compute(){
    LocalDate chrismas = LocalDate.of(2016, Month.DECEMBER, 25);
    LocalDate today = LocalDate.now();
    long days = DAYS.between(today, chrismas);
    System.out.println("There are " + days + " shopping days until Chrismas");
  }

  public static void main(String[] args){
    new BetweenDays().compute();
  }
}
```

*output*

```bash
There are 29 shopping days until Chrismas
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
    LocalDate chrismas = LocalDate.of(2016, Month.DECEMBER, 25);
    LocalDate today = LocalDate.now();
    long days = DAYS.between(today, chrismas);
    System.out.println("There are " + days + " shopping days until Chrismas");

    Period untilChristmas = Period.between(today, chrismas);
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
````

[Return to the main article](/techtalk/java)
