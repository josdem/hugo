+++
date = "2015-09-23T18:34:14-05:00"
draft = true
title = "Working with Strings"

+++

Groovy strings come in two flavors: plain strings and GStrings. Plain strings are instances of java.lang.String, and GStrings are instances of groovy.lang.Gstring which allows placeholder expressions to be resolved and evaluated at runtime.

| Start/End Characters  | Example                     |
| --------------------- | --------------              |
| Single quote          | 'hello josdem'              |
| Double quote          | "hello $name"               |
| Triple single quote   | ''' ======                  |
|                       |     ====== '''              |
| Triple double quote   | """ FirstName: $firstName   |
|                       |     LastName: $lastName """ |
| Forward slash         | /x(\d\*)y/                  |

* The single-quoted never pays any attention to placeholders. This is closely equivalent to Java string literals.
* The double-quoted form is the equivalent of the single-quoted form except if the text contains unescaped dollar signs, the dollar sign introduces a placeholder.
* The triple-quoted form  allows the literal to span several lines. Multiline string literals may also be GStrings, depending on whether single quotes or double quotes are used.
* The slashy form of string literal is also multiline but allow strings with backslashes to be specified simply without having to escape. This is particularly useful with regular expressions.

## GStrings Examples

**Abbreviated dollar syntax**

```groovy
def me = 'josdem'
def you = 'Jane'
def line = 'me $me, you $you!'

assert line == 'me josdem you Jane!'
```

**Extended dot syntax**

```groovy
class Person {
  def firstName
  def lastName
}

def person = new Person(firstName:'Jose Luis',lastName:'De la Cruz')
def line = "Person is: $person.firstName $person.lastName"
assert line == 'Person is: Jose Luis De la Cruz'
```

**Full syntax with braces**

```groovy
def date = new Date()
def timeZone = TimeZone.getTimeZone('CST')
def format = 'dd MMM YYYY HH:mm z'
def out = "Date is ${date.format(format, timeZone)}"

assert out == 'Date is 23 Sep 2015 20:41 CDT'
```

**Multiline GStrings**

```groovy
def flower1 = 'Roses'
def flower2 = 'Violets'
def color1 = 'red'
def color2 = 'blue'

def output = """$flower1 are $color1,
$flower2 are $color2,
Sugar is sweet,
And so are you."""

assert output == "Roses are red,
Violets are blue,
Sugar is sweet,
And so are you."
```

## String operations

```groovy
def greeting = 'Hello Groovy!'

assert greeting.startsWith('Hello')
assert greeting.replaceAll('Hello', 'Hola') == "Hola Groovy!"
assert greeting.getAt(0) == 'H'
assert greeting.indexOf('Groovy') == 6
assert greeting.contains('Groovy')
assert greeting[6..11] == 'Groovy'
assert 'Hi' + greeting - 'Hello' == 'Hi Groovy!'
assert greeting.count('o') == 3
assert 'x' * 3 == 'xxx'
```

**Left operator and StringBuffer**

```groovy
def greeting = 'Hello '
greeting <<= 'Groovy'
assert greeting instanceof java.lang.StringBuffer

greeting << '!'
assert greeting.toString() == 'Hello Groovy!'

greeting[1..4] = 'i'
assert greeting.toString() == 'Hi Groovy!'
```

**Spaceship operator**

Groovy adds some nice operators to the language. One of them is the spaceship operator. The operator is another way of referring to the compareTo method of the Comparable interface.

```groovy
assert -1 == ('a' <=> 'b')
assert 0 == (42 <=> 42)
```

Compare Persons

```groovy
class Person implements Comparable {
  String username
  String email

  int compareTo(other) {
      this.username <=> other.username
  }
}

assert 1 == (new Person([username:'josdem', email: 'joseluis.delacruz@gmail.com']) <=> new Person([username:'eric', email:'eric@email.com']))
```

[Return to the main article](/techtalk/groovy)
