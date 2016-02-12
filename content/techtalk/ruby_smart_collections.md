+++
date = "2016-02-12T12:48:09-06:00"
draft = true
title = "Smart Collections"

+++

If, in any method definition, you stick an asterisk before your parameter name, that parameter will soak up any extra arguments passed to the method. The value of the starred parameter will be an array containing all the extra arguments.

```ruby
beers = ['Calavaera', 'Lagrimas Negras', 'IPA']

def print_favorite_beers( name, *beers )
  puts "This are #{name}'s favorite beers:"
  beers.each{ |beer| puts beer }
end

print_favorite_beers('josdem', beers)
```

That is, you can send beers parameter with any size or you can omit it.

You can use hashes in case needs a collection with elements and does not care about types.

```ruby
hash = { :name => 'josdem', :number => 35 }

def print_person(person)
  person.each{ |name, number| puts "#{name} : #{number}"}
end

print_person(hash)
```


[Return to the main article](/techtalk/ruby)
