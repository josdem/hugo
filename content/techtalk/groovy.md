+++
date = "2015-08-29T14:03:12-05:00"
draft = true
title = "Groovy"

+++
Groovy is an optional typed, dynamic language for the Java platform with many features that are inspired by languages like Phyton, Ruby and Smalltalk, making them available to Java developers using Java-like syntax.

Working with Groovy feels like a partnership between you and the language, rather than a battle to express what is clear in your mind in a way the computer can understand.

* [Groovy and other languages](/techtalk/groovy_and_other_languages)
* Program_Structure
* Looping structures
* Closures
* Metaprogramming
* Design Patterns

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

## Examples

### Variables

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

### Output:

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
technologies.add("Groovy" )"
technologies << "Java" << "Griffon"
println technologies.contains("Java")

technologies.remove("Java")
println technologies
```

### Output

`true
[Groovy, Grails, Griffon]`

