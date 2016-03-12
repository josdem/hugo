+++
date = "2016-03-12T09:46:29-06:00"
draft = true
title = "Ruby Smart Strings"

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
string = %q{"Yehaw" the cowboy yelled, I'm not going!}
```

Ruby can span lines

```ruby
string = "one line
two lines
three lines"
```

[Return to the main article](/techtalk/ruby)

