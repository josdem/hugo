+++
date = "2016-06-13T16:49:12-05:00"
draft = true
title = "C++"

+++

* [Basics](/techtalk/c++_types)
* [Data Structures](/techtalk/c++_struct)
* [Loops and Collections](/techtalk/c++_loops)
* [Functions](/techtalk/c++_functions)
* [Exception Handling](/techtalk/c++_exception)
* [Classes](/techtalk/c++_classes)

## Compiling and Executing a program

Once we created a program we need to compile it. How your compile a program depends on your operating system and compiler. If we are using a command-line interface in a unix system we can compile it as follow:

```
$ g++ -o sum sum.cpp
```

Where -o is an argument to te compiler and names the file in which to put executable file. This command generates an executable file named `sum`, to execute in the current directory:

```
$ ./sum
```

Let's consider the following example:

```c++
#include <iostream>

int main()
{
  std::cout << "Enter two numbers" << std::endl;
  int v1=0, v2=0;
  std::cin >> v1 >> v2;
  std::cout << "The sum of " << v1 << " and " << v2 << " is " << v1 + v2 << std::endl;
  return 0;
}
```

iostream is a standard library that provides IO objects. To handle an input, we use an object `cin` (istream), for output we use an cout (ostream).

1. The first line tells the compiler that we want to use `iostream` library.
2. The first statement in the body of main executes an expression.

```c++
std::cout << "Enter two numbers" << std::endl;
```

3. The `<<` operator takes two operands: The left-hand operand must be an `ostream` object, the right operand is a value to print.
4. The `endl` is a special value called **manipulator**. Writting `endl` has the effect of ending the current line and flusing the buffer associated with that device.
5. The prefix `std::` indicates that the names `cout` and `endl` are defined inside the **namespace** named **std**. The namespaces allow us to avoid inadvertent collisions between the names.
6. The statement `std::cin >> v1 >> v2;` define analogously to the output operator. It takes an istream as its left-hand operand and an object as its right-hand operand. It reads data from the given `istream` and stores what was read in the given object.

## HelloWorld

So this is the Hello world example in C++

```c++
#include <iostream>

int main()
{
  std::cout << "Hello World!" << std::endl;
  return 0;
}
```

[Return to the main article](/techtalk/techtalks)

