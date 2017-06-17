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
def greeting = { "Hello $it!" }  // 1
assert greeting('Patrick') == 'Hello Patrick!'
```

1. Defines an implicit parameter, named it

```groovy
def list = [1,2,3,4,5,6,7,8]

list.each{
  print it  // 1
}
```

1. Prints each element from the list

```groovy
def number = 3
number.times {
  println "hi"  // 1
}
```

1. Prints hi three times

```groovy
def list = [1,2,3,4]
println list.findAll{
  it%2 == 0  // 1
}
```

1. For each element in the list is applied mod 2, in order to know the elements who remainder is zero

You may want to pass an closure as parameter

```groovy
def multiply(n, closure){
  closure(n)
}

assert 18 == multiply(3, { n -> n * 6 })
```

Other version of this code.

```groovy
def multiply(n, closure){
  closure(n)
}

assert 18 == multiply(3) { n -> n * 6 }
```

*Example*: Find the square of odds numbers > 0 until n.

```groovy
def square(n,closure){
  for(int i=1;i<=n;i++){
    closure(i)
  }
}

square(20, { if((it%2)==1) println "The square of $it is ${it*it}" })
```

*Output*

```
The square of 1 is 1
The square of 3 is 9
The square of 5 is 25
The square of 7 is 49
The square of 9 is 81
The square of 11 is 121
The square of 13 is 169
The square of 15 is 225
The square of 17 is 289
The square of 19 is 361
```

*Example:* A Palindrome is a word phrase, or number that reads the same backward or forward. Find palindrome words using closure

```groovy
def isPalindrome(word, closure){
  if(closure(word)){
    println ("$word is Palindrome")
  } else {
    println ("$word is NOT Palindrome")
  }
}

def closure =  { it == it.reverse() }

isPalindrome("Hello World", closure)
isPalindrome("anitalavalatina", closure)
```

Output:

```
Hello World is NOT palindrome
anitalavalatina is palindrome
```

## Scope

To fully understand the closures it's really important to understand the meaning of `this`, `owner` and `delegate`. In general:

* **this:** refers to the instance of the class that the closure was defined in.
* **owner:** is the same as this, unless the closure was defined inside another closure in which case the owner refers to the outer closure.
* **delegate:** is the same as owner. But, it is the only one that can be programmatically changed, and it is the one that makes Groovy closures really powerful.

Confused?. Me too, so let's take a look at this file called: `ClosureScope.groovy`

```groovy
class Owner {
  def closure = {
    assert this.class.name == 'Owner'               // 1
    assert delegate.class.name == 'Owner'           // 2
    def nestedClosure = {
      assert owner.class.name == 'Owner$_closure1'  // 3
    }
    nestedClosure()
  }
}

def closure = new Owner().closure
closure()
```


1. The `this` value always refers to the instance of the enclosing class.
2. Owner is always the same as this, except for nested closures.
3. Delegate is the same as owner by default, but it can be changed.

So how we can change delegate from a closure?, let's take a look at some code:

```groovy
class FirstClass {
  String myString = "I am first class"
}

class SecondClass {
  String myString = "I am second class"
}

class MainClass {
  def closure = {
    return myString
  }
}


def closure = new MainClass().closure
closure.delegate = new FirstClass()
assert closure() == 'I am first class'
```

Even when the delegate is set it can be change to something else, this means we can make the behavior of the closure dynamic.

[Return to the main article](/techtalk/groovy)
