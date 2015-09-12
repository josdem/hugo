+++
date = "2015-08-29T21:23:22-05:00"
draft = true
title = "Elixir"

+++
### Description
Elixir is a dynamic, functional language designed for building scalable and maintainable applications. Some of Elixir characteristics are:

* Uses Erlang VM
* Fault-tolerant
* Functional programming
* Easy to scale
* Easy to extend
* Immutable data types

### Basics
When you start interactive Elixir you can see someting like this:

```
Erlang/OTP 18 [erts-7.0] [source] [64-bit] [smp:4:4] [async-threads:10] [kernel-poll:false]
```

First you can see OTP Erlang version, Erlang was designed to run processes and that’s why is easy to scale, Erlang VM called BEAM seems to run as an operating system although Erlang VM runs as one OS process. OTP is an applications collection with Erlang modules, OTP provides libraries that Elixir uses. Erts stands for Erlang Runtime System Application. SMP detects processor core number. The asynchronous thread pool are OS threads which are used for I/O operations.

If you want quit from interactive Elixir type Ctrl + C or Ctrl + g to user switch command, as you can see follow options:

```
iex(1)>
User switch command
 --> h
  c [nn]            - connect to job
  i [nn]            - interrupt job
  k [nn]            - kill job
  j                 - list all jobs
  s [shell]         - start local shell
  r [node [shell]]  - start remote shell
  q                 - quit erlang
  ? | h             - this message
```

Then press h for menu and q for quit

### Hello World
```
$ cat simple.exs
IO.puts "Hello world from Elixir!"

$ elixir simple.exs
Hello world from Elixir!
```

* [Basic project in elixir](/techtalk/elixir_application)
* [Elixir and processes](/techtalk/elixir_processes)
* [FizzBuzz Example](/techtalk/elixir_fizzbuzz)

[Return to the main article](/techtalk/techtalks)


