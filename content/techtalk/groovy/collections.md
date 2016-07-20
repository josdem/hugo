+++
categories = ["techtalk", "code"]
date = "2016-07-20T16:08:01-05:00"
tags = ["josdem", "techtalks", "programming", "technology", "groovy", "collections", "groovy collections"]
title = "Collections"

+++

Groovy provides native support for various collection types, including lists, maps or ranges. Most of those are based on the Java collection types and decorated with additional methods.

**List**

```groovy

def list = []                            // 1
println list.class.name                  // 2
def list = ["One", "Two", "Three"]       // 3
list.each { print "$it " }               // 4

def list = []
list += 4                                // 5
println list                             // 6

def list = ['Java', 'Groovy']
list << 'Spring'                         // 7
println list                             // 8

def list = ['Java', 'Groovy']
list << 'Grails' << 'Spring'             // 9
println list                             // 10

def list = ['Java', 'Groovy']
list = list - 'Java'                     // 11
println list                             // 12

def list = ['Java', 'Groovy', 'Grails']
list = list - ['Java', 'Grails']         // 13
println list                             // 14

def list = ['Java', 'Groovy']
list = list - 'Kotlin'                   // 15
println list                             // 16
```

1. List definition
2. Prints java.util.ArrayList
3. Iniatilizing a list
4. Iterate through the list
5. Add an element to the list
6. Prints `[4]`
7. Add an element with leftShift
8. Prints `[Java, Groovy, Spring]`
9. Add two elements with leftShift
10. Prints `[Java, Groovy, Groovy, Spring]`
11. Removing an element
12. Prints `[Groovy]`
13. Removing elements defined in a list
14. Prints `[Groovy]`
15. Trying to remove an element that does not exist in the list and nothing happens
16. Prints `[Java, Groovy]`


[Return to the main article](/techtalk/groovy)
