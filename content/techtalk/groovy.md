+++
date = "2015-08-29T14:03:12-05:00"
draft = true
title = "Groovy"

+++
Groovy is an optional typed, dynamic language for the Java platform with many features that are inspired by languages like Phyton, Ruby and Smalltalk, making them available to Java developers using Java-like syntax.

Working with Groovy feels like a partnership between you and the language, rather than a battle to express what is clear in your mind in a way the computer can understand.

* [Groovy and other languages](/techtalk/groovy/groovy_and_other_languages)
* [Program Structure](/techtalk/groovy/program_structure)
* [Looping structures](/techtalk/groovy/looping_structures)
* [Closures](/techtalk/groovy/closures)
* [Metaprogramming](/techtalk/groovy/metaprogramming)
* [Design Patterns](/techtalk/groovy/design_patterns)
* [Kata String Calculator](/techtalk/groovy/kata_string_calculator)
* [Working with Strings](/techtalk/groovy/working_with_strings)
* [Rest client, headers and data](/techtalk/groovy/groovy_restclient)
* [Bindable](/techtalk/groovy/bindable)

## Characteristics

* Object-oriented programming language
* Dynamic Language
* Cross-platform
* Metaprogramming

## Features

* Write less code
* Easy to test
* Interface oriented
* High-level concurrency
* Wide-libraries support

## Variables

```groovy
/**
*   How variable assignment works
**/

def   x = 1
println x
x = 3.14159
println x
x = false
println x
x = "Groovy"
println x
```
*Output*

`1
3.14159
false
Groovy`

## Collections

```groovy
/**
*   Collections Management
**/

def technologies = ["Grails" ]
technologies.add("Groovy")
technologies << "Java" << "Griffon"
println technologies.contains("Java")

technologies.remove("Java")
println technologies
```

`output: true
[Groovy, Grails, Griffon]`

**Some funtions in collections**

```groovy
def list = [1,2,3]
def otherList = [3,4]

assert list.intersect(otherList) == [3]
assert 3 == list.max()
assert 1 == list.min()
```

```groovy
def list = ['Java', 'Groovy', 'Grails', 'Spring']

assert 1 == list.findIndexOf { it == 'Groovy' }
assert ['Grails', 'Groovy', 'Java', 'Spring'] == list.sort()
assert 'Groovy' == list.find { it == 'Groovy' }
assert ['Groovy', 'Grails'] == list.grep (~/G.*/)
assert 'Java,Groovy,Grails,Spring' == list.join(',')
```

```groovy
assert ['Groovy','Grails'] == [Language:'Groovy',Framework:'Grails'].collect{it.value}
assert ['Groovy','Grails'] == ['Groovy','Grails',null].findResults{it}
```

**Some collection operations**

```groovy
def vowels = ["A", "E", "I", "O", "U"]
def consonants = ["B", "C", "D", "F", "G", "H", "J", "K", "L", "N", "P", "Q", "R", "S", "T", "V", "W", "X", "Y", "Z"]
assert ["A", "E", "I", "O", "U", "B", "C", "D", "F", "G", "H", "J", "K", "L", "N", "P", "Q", "R", "S", "T", "V", "W", "X", "Y", "Z"] == vowels + consonants
```

[Return to the main article](/techtalk/techtalks)
