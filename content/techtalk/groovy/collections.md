+++
categories = ["techtalk", "code"]
date = "2016-07-20T16:08:01-05:00"
tags = ["josdem", "techtalks", "programming", "technology", "groovy", "collections", "groovy collections"]
title = "Collections"

+++

Groovy provides native support for various collection types, including lists, maps or ranges. Most of those are based on the Java collection types and decorated with additional methods.

## List

```groovy
def list = []                                         // 1
assert list.class.name == 'java.util.ArrayList'

def list = ["One", "Two", "Three"]                    // 2
list.each { print "$it " }                            // 3

def list = []
list.add(4)
assert list == [4]

def list = ['Java', 'Groovy']
assert list.contains('Java')

def list = ['Java', 'Groovy']
list.remove('Java')
assert list == ['Groovy']

def list = ['Java', 'Groovy']
list << 'Spring'                                       // 4
assert list == ['Java', 'Groovy', 'Spring']

def list = ['Java', 'Groovy']
list << 'Grails' << 'Spring'                           // 5
assert list == ['Java', 'Groovy', 'Grails', 'Spring']

def list = ['Java', 'Groovy']
list = list - 'Java'
assert list == ['Groovy']

def list = ['Java', 'Groovy', 'Grails']
list = list - ['Java', 'Grails']
assert list == ['Groovy']

def list = ['Java', 'Groovy']
list = list - 'Kotlin'                                 // 6
assert list == ['Java', 'Groovy']

def list = [0,1,2,3,4,5,6,7,8,9]
def subList = list[4..6]                               // 7
assert subList == [4, 5, 6]

def list = ['a', 'b', 'c', 'c']
assert list.toSet() == ['a','b','c'] as Set            // 8
```

1. List definition
2. Iniatilizing a list
3. Iterate through the list
4. Add an element with leftShift
5. Add two elements with leftShift
6. Trying to remove an element that does not exist in the list and nothing happens
7. We're creating a sublist defying a range as parameter
8. We can convert an array to a `Set`, We use the `toSet()` method to do this.

**Some functions in collections**

```groovy
def list = [1,2,3]
def otherList = [3,4]

assert list.intersect(otherList) == [3]
assert 3 == list.max()
assert 1 == list.min()

def list = ['Java', 'Groovy', 'Grails', 'Spring']

assert 1 == list.findIndexOf { it == 'Groovy' }
assert ['Grails', 'Groovy', 'Java', 'Spring'] == list.sort()
assert 'Groovy' == list.find { it == 'Groovy' }
assert ['Groovy', 'Grails'] == list.grep (~/G.*/)
assert 'Java,Groovy,Grails,Spring' == list.join(',')

assert [0,2,4,6] == (0..3).collect { it * 2 } // 1
```

1. `The collect()` method in Groovy can be used to iterate over collections and transform each element of the collection. The transformation is defined in as a closure and is passed to the `collect()`.

**Some collection operations**

```groovy
def vowels = ["A", "E", "I", "O", "U"]
def consonants = ["B", "C", "D", "F", "G", "H", "J", "K", "L", "M", "N", "P", "Q", "R", "S", "T", "V", "W", "X", "Y", "Z"]
assert ["A", "B", "C", "D", "E", "F", "G", "H", "I", "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V", "W", "X", "Y", "Z"] == (vowels + consonants).sort()
```

## Maps

```groovy
class Person {
  String firstName
  String lastName
}

Person p1 = new Person(firstName:'Eric', lastName:'de la Rosa')
Person p2 = new Person(firstName:'Jose Luis', lastName:'De la Cruz')

def map = ["eric":p1, "josdem":p2]
assert map.getClass().getName() == 'java.util.LinkedHashMap'
assert map.eric.lastName == 'de la Rosa'                               // 1
map.each { println "key: $it.key value: $it.value" }                   // 2

map.each { person ->                                                   // 3
  println "key: $person.key value: $person.value"
}

map.each { nickname, person ->                                         // 4
  println "key: $nickname vualue: $person"
}

println map.collect { nickname, person ->                              // 5
  "Person: $nickname: $person"
}

def result = map.find { nickname, person ->
  person.lastName.contains "Cruz"                                      // 6
}

assert result.key == 'josdem'
assert result.value instanceof Person

def result = map.findAll { nickname, person ->                         // 7
  person.lastName =~ /(D|d)e/
}

Output: [eric:Person@3fb041ae, josdem:Person@22457c6c]

// Some operations with Maps

assert ['Groovy','Grails'] == [Language:'Groovy',Framework:'Grails'].collect{it.value}
assert ['Groovy','Grails'] == ['Groovy','Grails',null].findResults{it}
```

1. Find a specific person in a map by key
2. Iterate over the map
3. Iterate using variable instead of $it
4. Iterate using variable for key and variable for value
5. Iterate and print unsing collect
6. Find an element with criteria in a map
7. Find a group of elements with criteria in a map

## Ranges

Ranges are lists with sequential values. Each range is also a list object, because Range extends `java.util.List`

```groovy
def range = (0..10)

assert range == [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
assert range.class.name == 'groovy.lang.IntRange'

def range = ('a'..'z')
assert range == [a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, y, z]
assert range.class.name == 'groovy.lang.ObjectRange'

Date today = new Date()
Date yesterday = today - 1
Date theDayBeforeYesterday = yesterdayi - 1
Date tomorroy = today + 1

println "Date as range"
def days = (theDayBeforeYesterday..tomorroy)
println days
```

Output:
Date as range
[Tue Jul 19 15:20:44 CDT 2016, Wed Jul 20 15:20:44 CDT 2016, Thu Jul 21 15:20:44 CDT 2016, Fri Jul 22 15:20:44 CDT 2016]

## Spread-Dot

The Groovy spread-dot operator is described as "equivalent to calling the collect method."

```groovy
def result = (1..10).collect{it * 2}                    // 1
def multiply = (1..10)*.multiply(2)
assert result =  [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
assert multiply =  [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]

def list = ['Java', 'Groovy', 'Grails']*.toUpperCase()  // 2
assert list == ['JAVA', 'GROOVY', 'GRAILS']

def range = (1..3)
assert [0,1,2,3] == [0,*range]                          // 3
def map = [a:1,b:2]
assert [a:1, b:2, c:3] == [c:3, *:map]
```

1. Multiply Each Item in a List by Two
2. It is on a collection of strings and toUpperCase() is used on each element of the collection via the spread-dot operator
3. Purpose is to extract entries from a collection and provide them as individual entries.

[Return to the main article](/techtalk/groovy)
