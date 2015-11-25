+++
date = "2015-11-25T11:05:36-06:00"
draft = true
title = "Elixir Practices"

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
