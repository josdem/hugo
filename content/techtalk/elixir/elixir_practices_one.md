+++
date = "2015-11-25T11:05:36-06:00"
title = "Elixir Practices"
categories = ["techtalk","code","elixir"]
tags = ["josdem","techtalks","programming","technology","elixir"]
description = "Create a function called sum that receives two integers and return their sum operation."
+++

1 - Create a function called sum that receives two integers and return their sum operation.

```elixir
sum = fn (a,b) -> a + b end
IO.puts sum.(1,2)
```

2 - Create a function that receives ([:a, :b], [:c, :d]) and returns [:a, :b, :c, :d]

```elixir
defmodule ListOperations do

  def list_concat(first_list, second_list) do
    first_list ++ second_list
  end

end

IO.inspect ListOperations.list_concat([:a,:b], [:c,:d])
```

3 - Create  function called pair_tuple_to_list that receives a {1234,5678} and return [1234,5678]

```elixir
defmodule ListOperations do

  def pair_tuple_to_list(tuple) do
    [elem(tuple,0), elem(tuple,1)]
  end

end

IO.inspect ListOperations.pair_tuple_to_list({1234,5678})
```

4 - The tuple returned by File.open has :ok as its first element if the file was opened, write a function that displays either the first line of successfully opened file or a simple error message if the file could not be opened.

```elixir
defmodule FileOperations do
  def handle_open({:ok, file}) do
    "Read data: #{IO.read(file, :line)}"
  end
  def handle_open({_, error}) do
    "Error: #{:file.format_error(error)}"
  end
end

IO.puts FileOperations.handle_open(File.open("hello.txt"))
IO.puts FileOperations.handle_open(File.open("not_exist.txit"))
```

This is the hello.txt content

```bash
Hello World!
```

5 - Functions can receive functions as parameters. In this example, apply is a function that takes a second function and a value. It returns the result of invoking that second function with the value as an argument.

```elixir
double = fn n -> n * 2 end
apply = fn (fun, value) -> fun.(value) end

IO.puts apply.(double,6)
```

6 - Enum module has a function called map. It takes two arguments: a collection and a function. It returns a list that is the result of applying that function to each element of the collection.

```elixir
list = [1,3,5,7,9]
double = fn n -> n * 2 end

IO.inspect Enum.map list, double
```

7 - The & Notation. The & operatonr converts the expression that follows into a function. Inside that expression, the placeholders &1, &2 and so on correspond to the first, second, and subsequent parameters of the function. Write a function receives a number and increments in one.

```elixir
add_one = &(&1 + 1)
IO.puts add_one.(45)
```

8 - Write a function using & notation to rewrite the following: `Enum.map [1,2,3,4], fn x -> x + 2 end`

```elixir
IO.inspect Enum.map [1,2,3,4], &(&1 + 2)
```

To download the project:

```bash
git clone https://github.com/josdem/elixir-workshop.git
git fetch
git checkout practices_one
```


[Return to the main article](/techtalk/elixir)
