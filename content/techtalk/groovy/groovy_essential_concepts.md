+++
categories = ["techtalk", "code"]
date = "2016-07-18T21:50:29-05:00"
tags = ["josdem", "techtalks", "programming", "technology", "Groovy", "Essential Concepts"]
title = "Essential Concepts"

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

[Return to the main article](/techtalk/groovy)
