+++
date = "2015-08-29T15:59:41-05:00"
draft = true
title = "Looping and FizzBuzz"

+++

Definying a range

```groovy
def range = ('a'..'z')
print range
```

`output: [a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, y, z]`

Iterate over a range of numbers:

```groovy
def message = ''

(1..5).each{
  message += 'HI '
}

assert message == 'HI HI HI HI HI '
```

The for loop in Groovy is much simpler in collections.

```groovy
["A","B","C"].each{                      // 1
  println it
}

def map = ['one':1, 'two':2, 'three':3]

map.each { key, value ->                 // 2
  println "$key : $value"
}

map.'four' = 4                          // 3

assert map['one'] == 1                  // 4

def text = "abc"
def list = []
for (c in text) {                       // 5
    list.add(c)
}
assert list == ["a", "b", "c"]
```

1. Iterate over a list
2. Iterate over a map
3. Adding a new value to the map
4. Getting an element from a map
5. Iterate over the characters in a string


Groovy supports the usual while

```groovy
def x = 0
def y = 5

while ( y-- > 0 ) {
    x++
}

assert x == 5
```

Example: Count letters from all Strings in a list

```groovy
def list = ['Java', 'Groovy', 'Grails']

Integer count = 0
list.each {
  if(it.class.name == "java.lang.String"){
    return count += it.size()
  }
}

assert count == 16
```

Example: Same that the last example but using collect

```groovy
def list = ['Java', 'Groovy', 'Grails']

Integer count = list.collect {
  if(it.class.name == "java.lang.String"){
    it.size()
  }
}.sum()

assert count == 16
```

**Note:** Use collect when you need to operate over all elements in a list

## FizzBuzz in Groovy

Fizz buzz is a group word game for children to teach them about division. Players take turns to count incrementally, replacing any number divisible by three with the word "fizz", and any number divisible by five with the word "buzz".

Our final example is to test a starting round of fizz buzz from 1 to 100 as follow:

  *1, 2, Fizz, 4, Buzz, Fizz, 7, 8, Fizz, Buzz, 11, Fizz, 13, 14 FizzBuzz...*

```groovy
(1..100).each{
  println "${it%3 ?'':'Fizz'}${it%5 ?'':'Buzz'}" ?: it
}
```

[Return to the main article](/techtalk/groovy)

