+++
date = "2016-02-10T16:32:01-06:00"
draft = true
title = "Ruby Control Structure"

+++

Ruby's most basic control structure: if statement

```ruby
if @writable
  @author = "josdem"
end
```

Consider, however, what would happen if we turned the logic around: If the document is read only, the natural tendency would be to simply throw in a ! or not::

```ruby
if not @read_only
  @author = "josdem"
end
```

An more idiomatic way would be:

```ruby
unless @read_only
  @author = "josdem"
end
```

The body of the statement is executed only if the condition is false.

In the same way we avoid negative conditions in `while` loops. Thus it is not:

```ruby
while ! parking_lot.is_full?
  park += 1
end
```

it's:

```ruby
until parking_lot.is_full?
  park +=1
end
```

Using modifier forms

Let's return to this snippet code:

```ruby
unless @read_only
  @author = "josdem"
end
```

We should collapse the whole thing into a single line like this:

```ruby
@author = "josdem" unless @read_only
```

You can also do similar things with `while`

```ruby
park +=1 while parking_lot.has_space?
```

**Using Each**

```ruby
beers = ['IPA', 'Lagrimas Negras', 'Calavera']

beers.each do |beer|
  puts beer
end
```

**Case Statement**

```ruby
framework = case language
when 'Groovy'
  puts 'Grails'
when 'Ruby'
  puts 'Rails'
else
  puts 'I Do Not Know'
end
```

This is equivalent and more compact:

```ruby
framework = case language
  when 'Groovy' then 'Grails'
  when 'Ruby' then 'Rails'
  else 'I Do Not Know'
```

Another expression-based way to make decision is the ternary operator.

```ruby
activity = programmer ? 'Java' : 'Futbol'
```

If the value of programmer is true, then the value of the whole expression is the thing between the question mark and colon, if the condition is false, the the expression evaluates the last part.

[Return to the main article](/techtalk/ruby)
