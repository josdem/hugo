+++
categories = ["techtalk", "code"]
date = "2016-07-18T20:29:41-05:00"
tags = ["josdem", "techtalks", "programming", "technology", "Groovy", "From Java to Groovy"]
title = "From Java to Groovy"

+++

You can convert a Java class to Groovy class in seven steps. Let's consider this example:

```java
public class HelloWorld {

  private String name;

  public String getName() {
    return name;
  }
  public void setName(String name) {
    this.name = name;
  }

  public String sayHello() {
    return "Hello " + this.name + "!";
  }

  public static void main(String[] args) {
    HelloWorld helloWorld = new HelloWorld();
    helloWorld.setName("josdem");
    System.out.println(helloWorld.sayHello());
  }
}
```

**Step One:**

  * Every class, method or field in Groovy is public at least you specify that is not.
  * ; is optional

```groovy
class HelloWorld {
  private String name

  String getName() {
    return name
  }

  void setName(String name) {
    this.name = name
  }

  String sayHello() {
    return "Hello " + this.name + "!"
  }

  static void main(String[] args) {
    HelloWorld helloWorld = new HelloWorld()
    helloWorld.setName("josdem")
    System.out.println(helloWorld.sayHello())
  }
}
```

**Step Two:**

Since Groovy has methods public, there is not necessary to specify set and get

```groovy
class HelloWorld {
  String name

  String sayHello() {
    return "Hello " + this.name + "!"
  }

  static void main(String[] args) {
    HelloWorld helloWorld = new HelloWorld()
    helloWorld.setName("josdem")
    System.out.println(helloWorld.sayHello())
  }
}
```

**Step Three:**

Groovy uses def to specify dynamic typing (DuckTyping) type check is defined at runtime

```groovy
class HelloWorld {
  def name

  def sayHello() {
    return "Hello " + this.name + "!"
  }

  static void main(def args) {
    def helloWorld = new HelloWorld()
    helloWorld.setName("josdem")
    System.out.println(helloWorld.sayHello())
  }
}
```

**Step Four:**

Groovy introduces placeholder using the dollar sign to specify a variable inside in a doube quoted string

```groovy
class HelloWorld {
  def name

  def sayHello() {
    return "Hello ${name}!"
  }

  static void main(def args) {
    def helloWorld = new HelloWorld()
    helloWorld.setName("josdem")
    System.out.println(helloWorld.sayHello())
  }
}
```

**Step Five:**

In Groovy the return keyword is optional, and the return value is the last expresion in a context (function)

```groovy
class HelloWorld {
  def name

  def sayHello() {
    "Hello ${name}!"
  }

  static void main(def args) {
    def helloWorld = new HelloWorld()
    helloWorld.setName("josdem")
    System.out.println(helloWorld.sayHello())
  }
}
```

**Step Six:**

The POJO (POGO in Groovy) has a constructor that accepts a Map so you can initialize an object with a map

```groovy
class HelloWorld {
  def name

  def sayHello() {
    "Hello ${name}!"
  }

  static void main(def args) {
    def helloWorld = new HelloWorld(name:'josdem')
    System.out.println(helloWorld.sayHello())
  }
}
```

**Step Seven:**

* Groovy supports scripting style
* You can short System.out.println to println
* Parenthesis are optional in there are at least one parameter

```groovy
class HelloWorld {
  def name

  def sayHello() {
    "Hello ${name}!"
  }
}

def helloWorld = new HelloWorld(name:'josdem')
println helloWorld.sayHello()
```

To download the project:

```bash
git clone https://github.com/josdem/groovy-techtalks.git
git fetch
git checkout feature/from-java-to-groovy
```

[Return to the main article](/techtalk/groovy)

