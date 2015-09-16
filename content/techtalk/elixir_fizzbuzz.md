+++
date = "2015-09-11T21:45:29-05:00"
draft = true
title = "Dojo Fizzbuzz With Test"

+++
Fizz buzz is a group word game for children to teach them about division. Players take turns to count incrementally, replacing any number divisible by three with the word "fizz", and any number divisible by five with the word "buzz".

Our Dojo goal is to test a starting round of fizz buzz from 1 to 10 as follow:

  *1, 2, Fizz, 4, Buzz, Fizz, 7, 8, Fizz, Buzz.*

Let's build our Dojo using Elixir testing. In order to do that we need a new project called fizzbuzz:

`
$ mix new fizzbuzz
`

Then add FizzBuzz logic in lib/fizzbuzz.ex file:

```
defmodule Fizzbuzz do

  def fizzbuzz(x) when is_number(x) do
    if rem(x,5) == 0 && rem(x,3) == 0 do
      "FizzBuzz"
      else if rem(x,5) == 0 do
        "Buzz"
        else if rem(x,3) == 0 do
          "Fizz"
          else
            x
        end
      end
    end
  end

end
```

Now is time to write our test cases; open test/fizzbuzz_test.exs and add this code:

```
defmodule FizzbuzzTest do
  use ExUnit.Case, async: true
  import Fizzbuzz


  test "When 1 then 1" do
    assert fizzbuzz(1) == 1
  end

  test "When 2 then 2" do
    assert fizzbuzz(2) == 2
  end

  test "When 3 then Fizz" do
    assert fizzbuzz(3) == "Fizz"
  end

  test "When 4 then 4" do
    assert fizzbuzz(4) == 4
  end

  test "When 5 then 5" do
    assert fizzbuzz(5) == "Buzz"
  end

  test "When 6 then 6" do
    assert fizzbuzz(6) == "Fizz"
  end

  test "When 7 then 7" do
    assert fizzbuzz(7) == 7
  end

  test "When 8 then 8" do
    assert fizzbuzz(8) == 8
  end

  test "When 9 then Fizz" do
    assert fizzbuzz(9) == "Fizz"
  end

  test "When 10 then Fuzz" do
    assert fizzbuzz(10) == "Buzz"
  end


end

```

Finally run all test with this command:

`
$ mix test
`

As result you should see this output.

```
..........

Finished in 0.08 seconds (0.08s on load, 0.00s on tests)
10 tests, 0 failures

Randomized with seed 521475
```

### Refactoring fizzbuzz module

Using matchers we can simplify our code.

* fizzbuzz(x) function will call private functions calculate_fizzbuzz using matchers
* _ character accepts any value to do the match
* Each private funcion do different action depending matchers

```
defmodule Fizzbuzz do
  def fizzbuzz(x) when is_number(x) do
    calculate_fizzbuzz(rem(x, 5), rem(x, 3), x)
  end

  defp calculate_fizzbuzz(0, 0, _), do: "Fizzbuzz"

  defp calculate_fizzbuzz(0, _, _), do: "Buzz"

  defp calculate_fizzbuzz(_, 0, _), do: "Fizz"

  defp calculate_fizzbuzz(_, _, x), do: x
end
```

After this refactor you can run the tests again.

[Return to the main article](/techtalk/elixir)
