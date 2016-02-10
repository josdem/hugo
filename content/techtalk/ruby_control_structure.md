+++
date = "2016-02-10T16:32:01-06:00"
draft = true
title = "Ruby Control Structure"

+++

Ruby's most basic control structure: if statement

```ruby
@writable = true
@auhor = "someone"

if @writable
  @author = "josdem"
end
```

Consider, however, what would happen if we turned the logic around: If the document is read only, the natural tendency would be to simply throw in a ! or not::

```ruby
@read_only = true
@auhor = "someone"

if not @read_only
  @author = "josdem"
end
```

An more idiomatic way would be:

```ruby
@read_only = false
@auhor = "someone"

unless @read_only
  @author = "josdem"
end
```

The body of the statement is executed only if the condition is false.


[Return to the main article](/techtalk/ruby)
