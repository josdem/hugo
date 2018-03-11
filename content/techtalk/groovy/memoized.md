+++
date = "2018-03-11T16:12:35-05:00"
tags = ["josdem", "techtalks","programming","technology","groovy","memoized","cache"]
categories = ["techtalk", "code", "groovy"]
title="Groovy Memoized"
+++

Memoized is useful when you need to create a cache for the results of the execution of the annotated method. Therefore it can be used anywhere you need to store previously calculated data based on specific input.

In order to review how it is working, lets review this piece of code to compute [Fibonacci sequence](https://en.wikipedia.org/wiki/Fibonacci_number). Basically it is characterized by the fact that every number after the first two is the sum of the two preceding ones:

```bash
 1, 1, 2, 3, 5, 8, 13, 21, 34, 55 ...
```

[Return to the main article](/techtalk/groovy)
