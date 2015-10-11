+++
date = "2015-10-11T10:54:41-05:00"
draft = true
title = "The Beam"

+++

Beam is the Erlang virtual machine, by default runs one OS thread per processor core to achieve maximum performance. BEAM has mechanism to communicate with other processes outside VM and it uses ports. BEAM does not have JIT compiler.

## Important aspects

* BEAM create process and they are isolated
* Process send messages to other processes as communication mechanism
* BEAM create an scheduler per core processor
* Each scheduler is associated with a thread in the OS.
* Each scheduler has an execution queue
* An important function in any scheduler is load balancing
* Each process has 2000 executions as life-cycle by default
* Each process has 2k at the beggining
* An Erlang process is not an OS process

## Memory and process

Memory in process management in Erlang has the following aspects:

* **Heap:** Each process has an heap memory section
* **ETS Tables:** Each process has ETS tables these provide the ability to store very large quantities of data and the current default limit is approximately 1400 tables. They are not implicitly garbage collected.
* **Atom Table:** The Erlang VM stores all the atoms defined in all the modules in a global atom table. An atom is basically a named constant as an enum. Atoms are never garbage collected and the atom table has a fixed size, system crashed when full. [Read this for more information](https://erlangcentral.org/wiki/index.php?title=Atom_Table)
* **Large binary space:** Large binaries definition in Erlang means (>64 bytes) and these data is stored in a separate area, a pointer is returned as reference.

## Garbage Collection

Erlang tried to solve was creating a platform for implementing Soft Realtime systems with a high level of responsiveness. Such systems require a fast Garbage Collection mechanism that doesn't stop the system from responding in a timely manner.

Erlang process which can be divided into three main parts: Process Control Block, Stack and Heap. It is so similar to Unix process memory layout.

* **PCB:** Process Control Block holds some information about the process such as its identifier (PID) in Process Table, current status (running, waiting), its registered name, the initial and current call, and also PCB holds some pointers to incoming messages which are members of a Linked List that is stored in heap.

* **Stack:** It is a downward growing memory area which holds incoming and outgoing parameters, return addresses, local variables and temporary spaces for evaluating expressions.

* **Heap:** It is an upward growing memory area which holds physical messages of process mailbox, compound terms like Lists, Tuples and Binaries and objects which are larger than a machine word such as floating point numbers. Binary terms which are larger than 64 bytes are not stored in process private heap.


The GC for private heap is generational. Generational GC divides the heap into two segments: young and old generations. This separation is based on the fact that if an object survives a GC cycle the chances of it becoming garbage in short term is low. So the young generation is for newly allocated data, and old generation is for the data that have survived an implementation specific number of GC.

[Return to the main article](/techtalk/elixir)
