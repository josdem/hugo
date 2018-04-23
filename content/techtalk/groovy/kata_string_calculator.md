+++
date = "2015-09-18T19:10:06-05:00"
title = "Kata String Calculator"
categories = ["techtalk", "code","groovy"]
tags = ["josdem", "techtalks", "programming", "technology", "Groovy", "Essential Concepts"]
description = "Create a function add() that take a string and return a integer with sum as result."
+++

This is the main goal for this kata:

1. A method receive an empty string and return 0
2. A method receive "1,2" as string and return 3
3. A method receive "1,2,3" as string and return 6
4. A method receive "1,2,3\n4" as string and return 10
5. Refactor your code


## Solution

**Step one: create testStringCalculator.groovy in any directory and add this code:**

```groovy
def add(args){
  return 0
}

assert add("") == 0
```

Type this command:

```groovy
groovy testStringCalculator.groovy
```

Be sure you have groovy installed in your computer.

**Step two: Write next test and then modify your code, the trick is add the minimum code necessary to pass the test**

```groovy
def add(args){
  if(args == "")
    0
  else
    3
}

assert add("") == 0
assert add("1,2") == 3
```

**Step three: Here is when we can not go on with the same pattern solution, so we improve our code with another algorithm**

```groovy
def add(args){
  if(args == "")
    return 0

  def list = args.split(",")
  def result = 0
  list.each {
    result += Integer.parseInt(it)
  }
  result
}

assert add("") == 0
assert add("1,2") == 3
assert add("1,2,3") == 6
```

**Step Four: So we add the last test and we modify our code in order to pass the test**

```groovy
def add(args){
  if(args == "")
    return 0

  args = args.replaceAll("\n",",")
  def list = args.split(",")
  def result = 0
  list.each {
    result += Integer.parseInt(it)
  }
  result
}

assert add("") == 0
assert add("1,2") == 3
assert add("1,2,3") == 6
assert add("1,2,3\n4") == 10
```

**Step Five: We realize that our code is pretty ugly, so needs a refactor, and we come with a new solution**

```groovy
def add(args){
  args.split("\n|,")?.collect { n ->
    n.isInteger() ? n as Integer : 0
  }.sum() ?: 0
}

assert add("") == 0
assert add("1,2") == 3
assert add("1,2,3") == 6
assert add("1,2,3\n4") == 10
```

[Return to the main article](/techtalk/groovy)
