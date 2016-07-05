+++
categories = ["algorithms", "pseudocode"]
date = "2016-07-04T21:00:30-05:00"
tags = ["josdem", "techtalks"]
title = "Algorithms"

+++

## Most pupular in the array

Assume I have an array that looks like the following:

```groovy
def array = [34,31,34,56,12,35,24,34,69,18]
```

Write a function that can determine what is the most popular in the array, in this case "34" because it is the number that appears the most often.

```groovy
Integer mostPopular(def array){
  Integer occurrences = 0
  Integer mostPopular = 0
  array.each{
    if(array.count { it } > occurrences){
      occurrences = array.count { it }
      mostPopular = it
    }
  }
  mostPopular
}
```

[Return to the main article](/techtalk/algorithms)
