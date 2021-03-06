+++
title = "Program structure"
date = "2015-08-29T14:33:15-05:00"
categories = ["techtalk", "code", "groovy"]
tags = ["josdem", "techtalks","programming","technology","groovy","memoized","cache"]
description = "Groovy classes must specify their package before the class definition. As Java does."
+++

**Package names**

They allows us to separate the code base without any conflicts. Groovy classes must specify their package before the class definition.

```groovy
// defining a package named com.yoursite
package com.project
```

**Imports**

In order to refer to any class you need a qualified reference to its package. Groovy provides several builder classes, such as MarkupBuilder. MarkupBuilder is inside the package groovy.xml so in order to use this class, you need to import it as shown:

```groovy
// importing the class MarkupBuilder
import groovy.xml.MarkupBuilder

// using the imported class to create an object
def xml = new MarkupBuilder()

assert xml != null
```

**Classes**

Classes in Groovy starts with class then a name and boby between braces.

```groovy
class Server {
  String toString() {
    "a server"
  }
}
```

In classes you can define variables at class level

```groovy
class Server {
  String name
  Cluster cluster
}
```

Initializing beans with named parameters and the default constructor.

```groovy
def server = new Server()
server.name = "Obelix"
server.cluster = aCluster
```

**A complete basic class**

Here is a class with a main function that allows you to execute it as Groovy program.

```groovy
package com.josdem

  class Todo{
    String name
    String note

    static main(String[] args){
      Todo todo = new Todo(name:"Tweet",note:"About my site")
      print todo.dump()
    }
}
```

**Groovy Truth**

All objects can be 'coerced' to a boolean value: everything that’s null, equal to zero, or empty evaluates to false, and if not, evaluates to true.

```groovy
def var
assert !var

var = 0
assert !var

var = []
assert !var

def name = 'josdem'
if (name) {
  println "hello $name"
}
```

**Safe graph navigation**

Groovy supports a variant of the . operator to safely navigate an object graph.

```groovy
println order?.customer?.address
```

Nulls are checked throughout the call chain and no NullPointerException will be thrown if any element is null, and the resulting value will be null if something’s null.

**Elvis operator for default values**

The Elvis operator is a special ternary operator shortcut which is handy to use for default values.

```groovy
def name = 'josdem' //1
def result = name!=null ? name : "Unknown"  //2
result = name ?: "Unknown" //3
```

**Note:** Line 2 and 3 are equivalents

Another expression-based way to make decision is the ternary operator.

```groovy
def programmer = true
def activity = programmer ? 'Java' : 'Futbol'
```

If the value of programmer is true, then the value of the whole expression is the thing between the question mark and colon, if the condition is false, the the expression evaluates the last part.

**Catch any exception**

If you don’t really care of the exception which are thrown inside your try block, you can simply catch any of them and simply omit the type of the caught exception. So instead of catching the exceptions like in:

```groovy
try {
  // ...
} catch (Exception t) {
  // something bad happens
}
```

**Annotations**

You can easily implement toString method in Groovy using annotations as follow:

```groovy
@groovy.transform.ToString
class User {
  String name
  String twitter
}

print new User(name:'Jose Luis',twitter:'@josdem')
```

`output: User(Jose Luis, josdem)`

Another example is TupleConstructor to get parameters as tuple and create our object by constructor

```groovy
@groovy.transform.ToString
@groovy.transform.TupleConstructor
class User {
  String name
  String twitter
}

print new User('Jose Luis','josdem')
```

`output: User(Jose Luis, josdem)`

[Return to the main article](/techtalk/groovy)
