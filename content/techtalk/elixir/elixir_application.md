+++
date = "2015-08-29T21:29:11-05:00"
draft = true
title = "Writting basic project in Elixir"

+++

Let’s create a basic project scructure by invoking mix new from the command line. Type:

```bash
$ mix new greet
```

Where greet is our name project. You should see following out put.

```bash
* creating README.md
* creating .gitignore
* creating mix.exs
* creating config
* creating config/config.exs
* creating lib
* creating lib/greet.ex
* creating test
* creating test/test_helper.exs
* creating test/greet_test.exs
```

There is a lib folder which contains all application code. mix.exs holds the metadata and dependencies of your application.

Now edit the file: lib/greet.ex and add this code:

```elixir
defmodule Greet do
  def main(_args) do
     IO.puts "Hello World"
  end
end
```

Elixir uses escript to build an executable. At first we need to set the main_module in mix.exs:

```elixir
def project do
  [app: :greet,
   version: "0.0.1",
   elixir: "~> 1.0",
   escript: [main_module: Greet], # <- add this line
   build_embedded: Mix.env == :prod,
   start_permanent: Mix.env == :prod,
   deps: deps]
end
```

Then create an executable and run it:

```bash
$ mix escript.build
$ ./greet
```

[Return to the main article](/techtalk/elixir)
