+++
date = "2015-08-29T16:18:31-05:00"
title = "Groovy and design patterns"
categories = ["techtalk", "code","groovy"]
tags = ["josdem", "techtalks", "programming", "technology", "groovy", "collections"]
description = "A design pattern is a general reusable solition to commonly occurring problem with a given context in software design. With Groovy is easier to implement design patterns than other languages such as C++ or Java. Using design patters we can share to the development teams a way to communicate solitions in the software design."
+++

A design pattern is a general reusable solition to commonly occurring problem with a given context in software design. With Groovy is easier to implement design patterns than other languages such as C++ or Java. Using design patters we can share to the development teams a way to communicate solitions in the software design.

## Abstract factory
Abstract out the creation of an object from its implementations. Let’s consider the following example:

```groovy
class Book {
  String title
  Integer pages
}

def create(clazz, properties){
  def instance = clazz.newInstance() // 1

  properties.each { name, value ->
    instance."$name" = value         // 2
  }

  instance                           // 3
}

assert create(Book, [title: 'Who moved my cheese?']) instanceof Book
```

1. Create a new object based on it’s type
2. Iterates the properties and assign a value from the map to the instance field
3. Return the new instance created

So now if we add a new class, let’s say CD that contains same title field but an Integer called volume, we can use the same factory to create our object.

```groovy
class Book {
  String title
  Integer pages
}

class CompactDisc {
  String title
  Integer volume
}

def create(clazz, properties){
  def instance = clazz.newInstance()

  properties.each { name, value ->
    instance."$name" = value
  }

  instance
}

assert create(CompactDisc, [title: 'Tri-state', volume: 1]) instanceof CompactDisc
```

## Strategy
Enables an algorithm behavior to be selected at runtime.

```groovy
def prices = [10, 20, 30, 40, 50]

def computeTotal(prices) {
  def total = 0
  for (price in prices){
    total += price
  }
  total
}

assert 150 == computeTotal(prices)
```

This code totalize all prices defined in the prices collection. But what if we want to get all prices under 35?, consider next code addition

```groovy
def prices = [10, 20, 30, 40, 50]

def computeTotal(prices) {
  def total = 0
  for (price in prices){
    total += price
  }
  total
}

def computeTotalUnder35(prices) {
  def total = 0
  for (price in prices){
    if (price < 35)
      total += price
  }
  total
}

assert 150 == computeTotal(prices)
assert 60 == computeTotalUnder35(prices)
```

This code definitely works, but goes against all good developer manners since we are duplicating code. And what if we want totalize all prices over 35, we should do copy and paste again? Using strategy pattern to this case we can get the following piece of code.

```groovy
def prices = [10, 20, 30, 40, 50]

def computeTotal(prices, closure) {
  def total = 0
  for (price in prices){
    if (closure(price))
      total += price
  }
  total
}

assert 150 == computeTotal(prices, { true })
assert 60 == computeTotal(prices, { it < 35 })
assert 90 == computeTotal(prices, { it > 35 })
```

Selector decides whether we should sumarize all prices or not, and if not what condition applies to sumarize them.

## Iterator pattern
Is so common pattern that we are using it all the time while programming in Groovy.

```groovy
def names = ["Joe", "Jane", "Jill"]

names.each(){
  println it
}
```

What if we want to print an index while iterate our collection?. We can solve this problem using this.

```groovy
def names = ["Joe", "Jane", "Jill"]

names.eachWithIndex(){ name, i ->
  println "$i. $name"
}
```

And if we want to print number of charactes in each element in the collection.

```groovy
def names = ["Joe", "Jane", "Jill"]

println names.collect { it.length() }
```

The collect() method in Groovy can be used to iterate over collections and transform each element of the collection. If we want to get names from the collection with four letters length?

```groovy
def names = ["Joe", "Jane", "Jill"]

println names.findAll { it.length() == 4 }
```

All programmers have done this once in their life: Iterate a collection and print out all elements in the collection with a comma as separator.

```groovy
def names = ["Joe", "Jane", "Jill"]

println names.join(', ')
```

## Delegation pattern
Delegation is better than inheritance since minimize dependency. Let’s see how Groovy manage delegation.

```groovy
class Worker {
  def work(){ 'working...' }
}

class Expert {
  def work() { 'Expert working...' }
  def analyze() { 'analyzing...' }
}

class Manager {
  @Delegate Worker worker = new Worker()
  @Delegate Expert expert = new Expert()
}

def manager = new Manager()
assert 'working...' == manager.work()
assert 'analyzing...' == manager.analyze()
```

What happening here is Manager in compiling time add all Worker methods in to the Manager class and all methods from the Expert class but only those methods that are not included so far.

To download the project:

```bash
git clone https://github.com/josdem/groovy-design-patterns.git
```

To run the project.

```
gradle test
```


[Return to the main article](/techtalk/groovy)
