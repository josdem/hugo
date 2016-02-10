+++
date = "2016-02-10T16:32:01-06:00"
draft = true
title = "Ruby Control Structure"

+++

Ruby's most basic control structure: if statement

```ruby
@readable = true

if @readable
  puts "This document is redable"
end
```

Consider, however, what would happen if we turned the logic around: If the document is not readable, the natural tendency would be to simply throw in a ! or not::

```ruby
@readable = false

if not @readable
  puts "This document is not redable"
end
```

[Return to the main article](/techtalk/ruby)
