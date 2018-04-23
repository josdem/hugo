+++
date = "2015-11-16T14:16:54-06:00"
title = "Elixir Essentials"
categories = ["techtalk","code","elixir"]
tags = ["josdem","techtalks","programming","technology","elixir"]
description = "Let's consider the following aspect, Unix utilities can be combined in ways undreamed of by their authors. And each one multiplies the potential of the others. A command pipeline can operate in parallel."
+++

Let's consider the following aspect, Unix utilities can be combined in ways undreamed of by their authors. And each one multiplies the potential of the others. A command pipeline can operate in parallel. If I write:

```bash
$ grep Elixir *.pml | wc -l
```

The world-count program, wc, runs at the same time as the grep command. Because wc consumes grep's output as it is produced, the answer is ready with virtually no delay once grep finishes.

Elixir lets us solve the problem in the same way the Unix shell does. Rather tha have command-line utilities, we have functions. And we can string them together as we please. The smaller more focused those functions, the more flexibility we have comining them.

## Pattern Matching

* Binds values to variables
* Matching handles structured data
* _ lets us ignore a match

In Elixir, the equals sign is not an assignment. Instead is like an assertion. It succeeds if Elixir can find a way of making the left-hand side equal the right-hand side.

```elixir
iex> a = 1
1
iex> 1 = a
1
iex> 2 = a
** (MatchError) no match of right hand side value: 1
```

Another example with lists

```elixir
iex> list = [1,2,3]
[1, 2, 3]
iex> [a,b,c] = list
[1, 2, 3]
iex> a
1
iex> b
2
iex> c
3
```

Elixir looks for a way to make the value of the left side the same as on the right. The left side is a list containing three variables, and the right is a list of three values, so the two sides could be made the same by setting the variables to the corresponing values. Elixir calls this process *pattern matching*

If we didn't need to capture a value during the match, we could use the special variable _ (underscore).

```elixir
iex> [1, _, _] = [1,2,3]
[1, 2, 3]
iex> [1, _, _] = [1,"cat","dog"]
[1, "cat", "dog"]
```

## Immutability

In Elixir, all values are immutable. The most complex nested list, the database records, there things behave just like the simplest integer. Their values are all immutable.
In Elixir, once a variable references a list such as `[1,2,3]`, you know it will always reference those same values (until you rebind the variable). And this makes concurrency a lot less frightening.

Once you accept the concept, coding with immutable data is surprisingly easy. You just have to remember that any function that transforms data will return a new copy of it. Thus, we never capitalize a sring. Instead, we return a capitalized copy of a string.

```elixir
iex> name = "elixir"
"elixir"
iex> cap_name = String.capitalize name
"Elixir"
iex> name
"elixir"
```

## Collections

* Tuples
* Lists
* Maps

A tuple is an ordered collection of values. As with all Elixir data structures, once created a tuple cannot be modified.

```elixir
iex> {status, count, action} = {:ok, 42, "next"}
{:ok, 42, "next"}
iex> status
:ok
iex> count
42
iex> action
"next"
```

Elixir has some operators that work specifically on lists:

```elixir
iex> [1,2,3] ++ [4,5,6]
[1, 2, 3, 4, 5, 6]
iex> [1,2,3,4] -- [2,4]
[1, 3]
iex> 1 in [1,2,3,4]
true
iex> 5 in [1,2,3,4]
false
```

A map is a collection key/value pairs. A map literal looks like this:

```elixir
iex> %{:one => 1, :two => 2}
%{one: 1, two: 2}
```

You can get values from map using the key.

```elixir
iex> states = %{"AL" => "Alabama", "WI" => "Wisconsin"}
%{"AL" => "Alabama", "WI" => "Wisconsin"}
iex> states["AL"]
"Alabama"
```

## Truth

Elixir has three special values related to Bolean operations: true, false and nil is treated as false in Boolean contexts.

**Comparison operators**

```elixir
a === b # strict equality
a !== b # strict inequality
a == b  # equality
a != b  # inequiality
a > b   # normal comparison
a >= b
a < b
a <= b
a or b  # true if a is true, otherwise b
a and b # false if a is false, oherwise b
not a   # false if a is true, true otherwise
a || b  # a if a is truthy, otherwise b
a && b  # b if a is truthy, otherwise a
!a      # false if a is truthy, otherwise true
```

[Return to the main article](/techtalk/elixir)
