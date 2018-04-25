+++
title = "Ruby Smart Strings"
categories = ["techtalk", "code","ruby"]
tags = ["josdem", "techtalks", "programming", "technology","ruby"]
date = "2016-03-12T09:46:29-06:00"
description = "You can define a single-quoted String escaped by a backslash, using placeholders, span lines."
+++

You can define a single-quoted String escaped by a backslash:

```ruby
a_string_with_a_quote = 'She said I\'m not going!'
```

You can put characters like tabs and newlines in a double-quoted string

```ruby
double_quoted_string = "This is a tab: \t, and this a newline: \n"
```

You can use placeholders for expressions

```ruby
author = 'Antoine de Saint-Exupéry'
title = 'The Little Prince'
puts "#{author} wrote #{title}"
```

Ruby offers a way to avoid backslash

```ruby
string = %q{"Yeehaw" the cowboy yelled, I'm not going!}
```

Ruby can span lines

```ruby
string = "one line
two lines
three lines"
```

Now that you have an string definition you can use some methods to manipulate it.

```ruby
# Gets string size
string.length

# Concatenates other_str to str
str + other_str

# Tests str and obj for equality
str == obj

# Returns true if str is empty
str.empty?

# Returns a copy of str with all occurrences of pattern replaced
str.gsub(pattern, replacement)

# Returns the index of the first occurrence of the given substring
str.index(substring [, offset])

# Returns a collection with strings delimited by ','
collection = "One, Two, Three".split(',')
```

[Return to the main article](/techtalk/ruby)

