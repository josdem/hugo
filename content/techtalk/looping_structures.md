+++
date = "2015-08-29T15:59:41-05:00"
draft = true
title = "Looping structures"

+++

Groovy supports the standard C for loop:

```groovy
String message = ''
for (int i = 0; i < 5; i++) {
    message += 'Hi '
}
assert message == 'Hi Hi Hi Hi Hi '
```

The for loop in Groovy is much simpler and works with any kind of array, collection, Map, etc.

```groovy
// iterate over a range
def x = 0
for ( i in 0..9 ) {
    x += i
}
assert x == 45

// iterate over a list
x = 0
for ( i in [0, 1, 2, 3, 4] ) {
    x += i
}
assert x == 10

// iterate over an array
def array = (0..4).toArray()
x = 0
for ( i in array ) {
    x += i
}
assert x == 10

// iterate over a map
def map = ['abc':1, 'def':2, 'xyz':3]
x = 0
for ( e in map ) {
    x += e.value
}
assert x == 6

// iterate over values in a map
x = 0
for ( v in map.values() ) {
    x += v
}
assert x == 6

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

[Return to the main article](/techtalk/groovy)

