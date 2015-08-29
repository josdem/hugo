+++
date = "2015-08-29T16:11:13-05:00"
draft = true
title = "Metaprogramming"

+++

### What is?

* Behavior changes at runtime
* Capabilities are introduced at runtime

### Groovy is dynamic
In Grails, for example, you see statements like Album.findByArtist('Oscar Peterson') but the Album class has no such method! Neither has any superclass. No class has such a method! The trick is that method calls are funneled through an object called MetaClass, which in this case regognizes that there’s no corresponding method in the bytecode of Album and therefore relays the call to its missingMethod handler. This knows about the naming convention of Grails' dynamic finder methods and fetches your favorite albums from the database.

### Expando
Dynamically expanding an object.

```groovy
def myExpando = new Expando()
myExpando.favoriteLanguage = 'Groovy'
myExpando.addNumbers = { i,j -> i + j }

assert 'Groovy' == myExpando.favoriteLanguage
assert 100 == myExpando.addNumbers(60,40)
assert myExpando.foo == null
```

Runtime mapping

```groovy
expando.favoriteLanguage = 'Groovy'
//maps to...
expando.setProperty('favoriteLanguage','Groovy')

expando.favoriteLanguage
//maps to...
expando.getProperty('favoriteLanguage')

expando.addNumbers(60,40)
//maps to...
expando.invokeMethod('addNumbers', [33,66] as Object[])
```

### Our own expando
With those 3 simple methods, we can create our own version of expando.

```groovy
package com.demo

import spock.lang.Specification

class MetaExpandoSpec extends Specification {
  void "should test property access"(){
  given:
    def expando = new MetaExpando() (1)
  when:
    expando.town = 'Mexico City'  (2)
  then:
    expando.town == 'Mexico City' (3)
  }
}
```

1. Creating an new instance of expando class
2. Assigning an value to a property does not exist
3. Asserting retrieve value is expected to be

If we run this test we will get an MissingPropertyException, since i’m assigning a value to a propery that it does not exist. So now we are going to add some code to the expando class that allow this work.

```groovy
package com.demo

class MetaExpando {

  private dynamicProps = [:] (1)

  void setProperty(String propName, val) {
    dynamicProps[propName] = val (2)
  }

  def getProperty(String propName){
    dynamicProps[propName] (3)
  }
}
```

1. Creating an empty map to storing properties values
2. Putting an entry to the map
3. Returning value in the map that is associated with the key

```groovy
void "should invoke a method does not exist"(){
  given:
    def expando = new MetaExpando()
  when:
    expando.addNumbers = { x, y, z -> x + y + z } (1)
  then:
    100 == expando.addNumbers(30, 20, 50) (2)
}
```

1. Define a clusure which receives three parameters, and summarize them.
2. We expect that sending 30, 20 and 50 as parameters the result will be 100

```groovy
package com.demo

class MetaExpando {

  private dynamicProps = [:]

  void setProperty(String propName, val) {
    dynamicProps[propName] = val
  }

  def getProperty(String propName){
    dynamicProps[propName]
  }

  def methodMissing(String methodName, args) { (1)
    def prop = dynamicProps[methodName] (2)
    if (prop instanceof Closure){ (3)
      return prop(*args) (4)
    }
  }
}
```

1. MethodMissing is called in groovy by default when no method is found in class.
2. Retrieving the map the value associated with the method name
3. Verify if value from the map is a closure
4. Execute closure and pass all the parameters

### Closure delegates

* Closure may be assignated a "delegate"
* Closures relay method calls to their delegate

Every closure has a delegate method associated with it. Let’s consider the following code.

```groovy
closure = {
  append 'one'
  append 'Two'
}

closure()
```

This code will thrown an groovy.lang.MissingMethodException since method append does not exist. But if we do something like this:

```groovy
closure = {
  append 'one'
  append 'Two'
}

def sb = new StringBuffer()
closure.delegate = sb
closure()

print "StringBuffer value: ${sb}"
```

The sb value is now 'oneTwo', since closure delegates the append functionality to the StringBuffer class. So when we use delegate, we are allowing to Groovy the oportunitty to delegate others respond to that method calls.

### Adding functionality at runtime

Sometimes you might want to change behavior to some functionality allowing to the developer write less code, let’s consider the following snipppet.

```groovy
String.metaClass.doSomething = {
  println "I'm doing something"
}

name = "josdem"
name.doSomething()
```

What is happening here is to say, when someone invoke doSomething method in the String class that by the way does not exist, do this.

[Return to the main article](/techtalk/groovy)
