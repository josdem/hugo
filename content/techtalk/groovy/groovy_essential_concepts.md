+++
title = "Essential Concepts"
date = "2016-07-18T21:50:29-05:00"
categories = ["techtalk", "code","groovy"]
tags = ["josdem", "techtalks", "programming", "technology", "Groovy", "Essential Concepts"]
description = "Groovy essential concepts, Assert, everything is an object, finders."
+++

**Assert** is a way to evaluate an expresion if it is `true`

```groovy
assert(true)
assert 1 == 1
def x = 1; assert x == 1
def y = 2; assert y == 2
assert ('text'*3<<'hello').size() == 4*3+5
```

Everything in Groovy is an object

```groovy
int a = 1
double b = 2.0
float c = 3.0
char d = 'a'
boolean e = true

println a.class
println b.class
println c.class
println d.class
println e.class
```

Output:

```
class java.lang.Integer
class java.lang.Double
class java.lang.Float
class java.lang.Character
class java.lang.Boolean
```

Groovy automatically import this packages:

```groovy
import java.lang.*
import java.util.*
import java.net.*
import java.io.*
import groovy.lang.*
import groovy.util.*
```

And this classes:

```groovy
import java.math.BigInteger
import java.math.BigDecimal
```

**Finders**

The `findXXX()` methods take a closure and if an element matches the condition defined in the closure we get a result. We can also use the `any()` method to verify if at least one element applies to the closure condition, or we use the `every()` method to verify all elements that confirm to the closure condition. Both the `any()` and `every()` method return a `Boolean` value.

```groovy
assert 'abcde'.find{ it > 'b' } == 'c'
assert 'abcde'.findAll{ it > 'b' } == ['c', 'd', 'e']
assert 'abcde'.findIndexOf{ it > 'c' } == 3
assert 'abcde'.every{ it < 'g' }
assert 'abcde'.any{ it > 'c' }
assert 'josdem'.replace('o','0') == 'j0sdem'
assert 'AbcdE'.equalsIgnoreCase('aBCDe')
```

**Matchers**

In Groovy we use the =~ operator (find operator) to create a new matcher object. We can use a second operator, ==~ (match operator), to do exact matches. With this operator the matches() method is invoked on the matcher object. The result is a Boolean value.

```groovy
def matcher = ('groovy' =~ /[a-z]*/)
assert matcher instanceof java.util.regex.Matcher
assert matcher[0] == 'groovy'

def match = ('groovy' ==~ /[a-z]*/)
assert match instanceof Boolean

def matcher =  'Groovy rocks!' =~ /Groovy/
assert matcher[0] == 'Groovy'

assert 'Groovy rocks!' ==~ /Groovy rocks!/
assert 'Groovy rocks!' ==~ /Groovy.*/

String uuid = "TimbreFiscalDigital version=\"1.0\" UUID=\"84918EDD-17BE-4726-A217-8430B97B0BD9\" FechaTimbrado=\"2016-04-05T14:48:40\" selloCFD=\"Rgk8ofr4Ud35k2=\" selloSAT=\"z9DT2PLPg9Kg="
def matcher = uuid =~ /[a-zA-Z0-9]{8}-[a-zA-Z0-9]{4}-[a-zA-Z0-9]{4}-[a-zA-Z0-9]{4}-[a-zA-Z0-9]{12}/
assert matcher[0] == "84918EDD-17BE-4726-A217-8430B97B0BD9"
```

**Dynamic Typing**

A language is dynamically typed if the type of a variable is interpreted at runtime.

```groovy
def var
var = 1; assert var.class == java.lang.Integer
var = 2f; assert var.class == java.lang.Float
var = 3d; assert var.class == java.lang.Double
var = 4g; assert var.class == java.math.BigInteger
var = 'Groovy'; assert var.class == java.lang.String
var = true; assert var.class == java.lang.Boolean
```

[Return to the main article](/techtalk/groovy)
