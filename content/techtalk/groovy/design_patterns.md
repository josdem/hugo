+++
date = "2015-08-29T16:18:31-05:00"
draft = true
title = "Groovy and design patterns"

+++
A design pattern is a general reusable solition to commonly occurring problem with a given context in software design. With Groovy is easier to implement design patterns than other languages such as C++ or Java. Using design patters we can share to the development teams a way to communicate solitions in the software design.

## Abstract factory
Abstract out the creation of an object from its implementations. Let’s consider the following example:

```groovy
class Book {
  String title
  Integer page
}

def create(klass, properties){
  def instance = klass.newInstance() (1)

  properties.each { name, value ->
    instance."$name" = value  (2)
  }

  instance (3)
}

println create(Book, [title: 'Who moved my cheese?'])
```

1. Create a new object based on it’s type
2. Iterates the properties and assign a value from the map to the instance field
3. Return the new instance created

So now if we add a new class, let’s say CD that contains same title field but an Integer called volume, we can use the same factory to create our object.

```groovy
class Book {
  String title
  Integer page
}

class CD {
  String title
  Integer volume
}

def create(klass, properties){
  def instance = klass.newInstance()

  properties.each { name, value ->
    instance."$name" = value
  }

  instance
}

println create(CD, [title: 'Tri-state', volume: 1]).dump()
```

## Strategy
Enables an algorithm behavior to be selected at runtime.

```groovy
def prices = [10, 20, 30, 40, 50]

def totalPrices(prices) {
  def total = 0
  for (price in prices){
    total += price
  }
  total
}

println totalPrices(prices)
```

This code totalize all prices defined in the prices collection. But what if we want to get all prices under 35?, consider next code addition

```groovy
def prices = [10, 20, 30, 40, 50]

def totalPrices(prices) {
  def total = 0
  for (price in prices){
    total += price
  }
  total
}

def totalPricesUnder35(prices) {
  def total = 0
  for (price in prices){
    if (price < 35)
      total += price
  }
  total
}

println totalPrices(prices)
println totalPricesUnder35(prices)
```

This code definitely works, but goes against all good developer manners since we are duplicating code. And what if we want totalize all prices over 35, we should do copy and paste again? Using strategy pattern to this case we can get the following piece of code.

```groovy
def prices = [10, 20, 30, 40, 50]

def totalPrices(prices, selector) {
  def total = 0
  for (price in prices){
    if (selector(price))
      total += price
  }
  total
}

println "Total prices:"  + totalPrices(prices, { true })
println "Prices under 35:"  + totalPrices(prices, { it < 35 })
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
  def work(){ println "working..." }
}

class Expert {
  def work() { println "expert working..." }
  def analyze() { println "analyzing..." }
}

class Manager {
  @Delegate Worker worker = new Worker()
  @Delegate Expert expert = new Expert()
}

def manager = new Manager()
manager.work()
manager.analyze()
```

Output

```
working...
analyzing...
```

What happening here is Manager in compiling time add all Worker methods in to the Manager class and all methods from the Expert class but only those methods that are not included so far.

[Return to the main article](/techtalk/groovy)
