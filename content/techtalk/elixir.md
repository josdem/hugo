+++
date = "2015-08-29T21:23:22-05:00"
title = "Elixir"
categories = ["techtalk","code","elixir"]
tags = ["josdem","techtalks","programming","technology","elixir"]
description = "Elixir is a dynamic, functional language designed for building scalable and maintainable applications."
+++

* [Basic project in Elixir](/techtalk/elixir/elixir_application)
* [Elixir and processes](/techtalk/elixir/elixir_processes)
* [FizzBuzz Example](/techtalk/elixir/elixir_fizzbuzz)
* [BEAM](/techtalk/elixir/elixir_beam)
* [Elixir Essentials](/techtalk/elixir/elixir_essentials)
* [Elixir Practices One](/techtalk/elixir/elixir_practices_one)

## Description
Elixir is a dynamic, functional language designed for building scalable and maintainable applications. Some of Elixir characteristics are:

* Uses Erlang VM
* Fault-tolerant
* Functional programming
* Easy to scale
* Easy to extend
* Immutable data types

## Basics
When you start interactive Elixir you can see someting like this:

```bash
$ iex

Erlang/OTP 18 [erts-7.0] [source] [64-bit] [smp:4:4] [async-threads:10] [kernel-poll:false]
```

First you can see OTP Erlang version, Erlang was designed to run processes and that’s why is easy to scale, Erlang VM called BEAM seems to run as an operating system although Erlang VM runs as one OS process. OTP is an applications collection with Erlang modules, OTP provides libraries that Elixir uses. Erts stands for Erlang Runtime System Application. SMP detects processor core number. The asynchronous thread pool are OS threads which are used for I/O operations.

If you want quit from interactive Elixir type Ctrl + g to user switch command, as you can see follow options:

```bash
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

Using mix command you can:

* Create a new project
* Compile
* Run tests
* Manage versions
* Create a release package
* Etc.

Type:

```bash
$ mix -h
```

For more information.

## Hello World
```bash
$ cat simple.exs
IO.puts "Hello world from Elixir!"

$ elixir simple.exs
Hello world from Elixir!
```

[Return to the main article](/techtalk/techtalks)


