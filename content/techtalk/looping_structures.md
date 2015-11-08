+++
date = "2015-08-29T15:59:41-05:00"
draft = true
title = "Looping structures"

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
// iterate over a list
["A","B","C"].each{
  println it
}

// iterate over a map
def map = [one:1, two:2, three:3]

for (item in map){
  println "key: ${item.key} value: ${item.value}"
}

//Map new value as simple as:
map.four = 4

// iterate over the characters in a string
def text = "abc"
def list = []
for (c in text) {
    list.add(c)
}
assert list == ["a", "b", "c"]
```

Groovy supports the usual while

```groovy
def x = 0
def y = 5

while ( y-- > 0 ) {
    x++
}

assert x == 5
```

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

