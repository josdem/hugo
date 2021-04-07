+++
title = "Java Time API"
categories = ["techtalk", "code","java"]
tags = ["josdem", "techtalks", "programming", "technology", "Java"]
date = "2016-11-26T10:46:26-06:00"
description = "Is an evolution from the previous java.util.Date (Calendar, TimeZone & DateFormat), instances of time/date now are immutable, time and Date operations now are thread safe, API support strong typing, which enables you to write better code."
+++

Some important things that we have to know about the `java.time API` is the following:

* Is an evolution from the previous `java.util.Date (Calendar, TimeZone & DateFormat)`
* Instances of time/date now are immutable. (This is importat for lambda expressions)
* Time and Date operations now are thread safe.
* The API support strong typing, which enables you to write better code.

*A quick glance to the Date methods*

```java
import java.time.*;

public class DateOperations {

  private void operations() throws Exception {
    LocalDate localDate = LocalDate.now();
    System.out.println("LocalDate in YYYY-MM-DD format: " + localDate);
    System.out.println("Day of the week: " + localDate.getDayOfWeek());
    System.out.println("Next week: " + localDate.plusDays(7));
    System.out.println("Is a Leap Year?: " + localDate.isLeapYear());

    LocalDate christmas = LocalDate.of(2020, Month.DECEMBER, 25);
    System.out.println("Today is before Christmas?: " + localDate.isBefore(christmas));

    LocalDate christmasParsed = LocalDate.parse("2020-12-25");
    System.out.println("Christmas Parsed: " + christmasParsed);

    LocalDateTime localDateTime = LocalDateTime.now();
    System.out.println("Local Date Time: " + localDateTime);

    ZonedDateTime zonedDateTime = ZonedDateTime.now();
    System.out.println("Zoned Date Time: " + zonedDateTime);

    Instant instant = Instant.now();
    System.out.println("Instant: " + instant);

    Instant start = Instant.now();
    Thread.sleep(1000);
    Instant end = Instant.now();
    System.out.println("Duration in seconds: " + Duration.between(start, end).getSeconds());
  }

  public static void main(String[] args) throws Exception {
    new DateOperations().operations();
  }
}
```

*output*

```bash
LocalDate in YYYY-MM-DD format: 2020-07-10
Day of the week: FRIDAY
Next week: 2020-07-17
Is a Leap Year?: true
Today is before Christmas?: false
Christmas Parsed: 2020-12-25
Local Date Time: 2020-07-10T13:25:42.392957
Zoned Date Time: 2020-07-10T13:25:42.393143-04:00[America/Detroit]
Instant: 2020-07-10T17:25:42.393278Z
Duration in seconds: 1
```

**Calculating Between Days**

When you want to perform calculations between days, temporal unit interface unit can be used, since represents a unit of time. Enumeration ChronoUnit actually implements our temporal unit interface. By importing `java.time.temporal,ChronoUnit.*` you can use enumerations such as DAYS, WEEKS, MONTHS, YEARS and you can use methods as between.

*example*

```java
import java.time.LocalDate;
import java.time.Month;
import java.time.Period;

import static java.time.temporal.ChronoUnit.DAYS;

public class BetweenDays {

  private void compute() {
    LocalDate christmas = LocalDate.of(2021, Month.DECEMBER, 25);
    LocalDate today = LocalDate.now();
    long days = DAYS.between(today, christmas);
    System.out.println("There are " + days + " shopping days until Christmas");

    Period untilChristmas = Period.between(today, christmas);
    System.out.println(
        "There are "
            + untilChristmas.getMonths()
            + " months and "
            + untilChristmas.getDays()
            + " days until Christmas");
  }

  public static void main(String[] args) {
    new BetweenDays().compute();
  }

```

*output*

```bash
There are 168 shopping days until Christmas
```

A period class also have a between method and you can use it to figure out how many days or months are between dates.

*example*

```java
import java.time.LocalDate;
import java.time.Month;
import java.time.Period;

import static java.time.temporal.ChronoUnit.DAYS;

public class BetweenDays {

  private void compute() {
    LocalDate christmas = LocalDate.of(2020, Month.DECEMBER, 25);
    LocalDate today = LocalDate.now();
    long days = DAYS.between(today, christmas);
    System.out.println("There are " + days + " shopping days until Christmas");

    Period untilChristmas = Period.between(today, christmas);
    System.out.println(
        "There are "
            + untilChristmas.getMonths()
            + " months and "
            + untilChristmas.getDays()
            + " days until Christmas");
  }

  public static void main(String[] args) {
    new BetweenDays().compute();
  }
}
```

*output*

```bash
There are 168 shopping days until Christmas
There are 5 months and 15 days until Christmas
```

**How to format LocalDate**

To format a LocalDate object, uses DateTimeFormatter

*example*

```java
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

public class DateFormatter {

    private void format(){
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MMM");
        LocalDate today = LocalDate.now();
        System.out.println("YYYY-MMM format from now is: " + today.format(formatter));
    }

    public static void main(String[] args){
        new DateFormatter().format();
    }

}
```

*output*

```bash
YYYY-MMM format from now is: 2020-Jul
```

To browse the code go [here](https://github.com/josdem/java-workshop), to download the code:

```bash
git clone git@github.com:josdem/java-workshop.git
cd date-and-time
```

[Return to the main article](/techtalk/java)
