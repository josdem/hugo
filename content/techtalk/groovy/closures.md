+++
date = "2015-08-29T16:05:19-05:00"
draft = true
title = "Closures"

+++
Informally, a closure can be recognized as a list of statements within braces, like any other code block. It optionally has a list of identifiers to name the parameters passed to it, with an → making the end of the list.

It’s easiest to undertands closures through examples, so let’s consider the following example:

```groovy
def adder = {i,j -> i + j}  // 1
assert adder(40,60) == 100  // 2
```

1. Define a closure which receive two parameters and then sum that parameters
2. Verify that calling adder closure with 40 and 60 as parameters returns 100

Let’s consider a slightly more complicated question: If n people are at a party and everyone clinks glasses with everybody else, how many clinks do you hear?.

To answer the question, you can use Integer’s upto method which does something for every Integer starting at the current value. You apply this method to the problem by imaginating people arriving at the party at one by one. As people arrive, they clink glasses with everyone who is already present. This way, everyone clinks glasses with everyone else exactly once.

```groovy
def totalClinks = 0
def partyPeople = 100

1.upto(partyPeople) { guestNumber ->
  clinksWithGuest = guestNumber - 1                        // 1
  totalClinks += clinksWithGuest                           // 2
}

assert totalClinks == (partyPeople * (partyPeople-1)) / 2  // 3
```

1. Counting number of people already present
2. Keep running total of the number of clinks
3. Test the result using Gauss' formula

Another simple examples of closures would be:

```groovy
def greeting = { "Hello, $it!" } (1)
assert greeting('Patrick') == 'Hello, Patrick!'
```

1. Defines an implicit parameter, named it

```groovy
list.each{
  print it (1)
}
```

1. Prints each element from the list

```groovy
def number = 3
number.times {
  println "hi" (1)
}
```

1. Prints hi three times

```groovy
def list = [1,2,3,4]
println list.findAll{
  it%2 == 0 (1)
}
```

You may want to pass an closure as parameter

```groovy
def multiply(n, closure){
  closure(n)
}

assert 18 == multiply(3, { n -> n * 6 })
```

Simplifying this code.

```groovy
def multiply(n, closure){
  closure(n)
}

assert 18 == multiply(3) { n -> n * 6 }
```

1. Go through the list and find all elements that is divisible by two

[Return to the main article](/techtalk/groovy)
