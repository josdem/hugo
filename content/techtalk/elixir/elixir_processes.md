+++
date = "2015-08-29T21:33:05-05:00"
title = "Elixir Processes"
categories = ["techtalk","code","elixir"]
tags = ["josdem","techtalks","programming","technology","elixir"]
description = "In Elixir, all code runs inside processes. Processes are isolated from each other, run concurrent to one another and communicate via message passing, they also provide the means for building distributed and fault-tolerant programs."
+++

In Elixir, all code runs inside processes. Processes are isolated from each other, run concurrent to one another and communicate via message passing, they also provide the means for building distributed and fault-tolerant programs.

Elixir manages concurrency in this way, since it can spawn process and send/receive messsages between them. Each process has its own queue, and each process can manage its own queue.

The basic mechanism for starting a process is by spawning, basically spawn/1 takes a function which it will execute in another process.

```bash
iex> spawn fn -> 1 + 2 end
#PID<0.43.0>
```

spawn/1 returns a PID (process identifier). We can send messages to a process with send/2 and receive them with receive/1. Let’s consider next example.

```bash
iex(1)> defmodule Messengine do
...(1)>  def message do
...(1)>    receive do
...(1)>      {:name, value} ->
...(1)>        IO.puts("Receiving message from = #{value}")
...(1)>    end
...(1)>  end
...(1)> end
```

We defined a Module with a message method which receive a tuple with atom :name and value as string, next thanks to receive method puts in the output console the message "Receiving message from ${parameter}"

```bash
iex(2)> pid = spawn(fn -> Messengine.message() end)
```

Next we spawned the function in a process and store as pid value.

```bash
iex(3)> send pid, {:name, "josdem"}
```

Finally we sent the tuple to the process pid and we should see the message as response in the console.

[Return to the main article](/techtalk/elixir)

