+++
date = "2016-06-14T17:35:25-05:00"
title = "C++ Basics"
categories = ["techtalk", "code","c++"]
tags = ["josdem", "techtalks", "programming", "technology","c++","cplusplus", "C++ basics"]
description = "C++ is a statically typed language; type checking is done at compile time."
+++

## Types

C++ is a statically typed language; type checking is done at compile time.

---
| Type           | Meaning                            | Minimum Size          |
|------          |-----------                         |--------------         |
| bool           | boolean                            | NA                    |
| char           | character                          | 8 bits                |
| wchar_t        | wide character                     | 16 bits               |
| char16_t       | Unicode character                  | 16 bits               |
| char32_t       | Unicode character                  | 32 bits               |
| short          | short integer                      | 16 bits               |
| int            | integer                            | 16 bits               |
| long           | long integer                       | 32 bits               |
| long long      | long integer                       | 64 bits               |
| float          | single-precision floating-point    | 6 significant digits  |
| double         | double-precision floating-point    | 10 significant digits |
| long double    | extended-precision floating-point | 10 significant digits |
---

A char is guaranted to be big enough to hold a basic character, wchar_t, w_char16_t and char32_t are used for extended character sets (unicode). The remaining integral types represent integer values of (potentially) different sizes. Except for bool and the extended character types, the integral types may be **signed** or **unsigned**. A signed type represents negative or positive numbers; an unsigned type represents only values greather than or equal to zero. The types `int, short, long, long long` are signed, we obtain the corresponding unsigned type by adding unsigned to the type.

## Type Aliases

A **type alias** is a name that is a synonym for another type. Type aliases let us simplify complicated type definitions, making those types easier to use.

```c++
typedef unsigned int size;
size border = 10;
```

size is a synonym for unsigned int

## References

A reference defines an alternative name for an object. We define a reference type by writting a declaration of the form &d, wherer id is the name being declared.

```c++
#include <iostream>

int main()
{
  int number = 1024;
  int &reference = number;

  std::cout << reference << std::endl;
}
```

`&reference` refers to number

## Pointers

A pointer is a compound type that "points to" another type. Like references, pointers are used fo inderect access to other objects. Unlike a reference, a pointer is an object in it's own right. Pointers can be assigned and copied; a single pointer can point to several different objects over its lifetime. Unlike a reference, a pointer need not be initialized at the time is defined.

```c++
#include <iostream>

int main()
{
  int number = 1024;
  int *pointer = &number;

  std::cout << "pointer address:" << pointer << std::endl;
}
```

**WARNING:** The address stored in a pointer can be a null pointer.

Otherwise, we can use * to access that object.

```c++
#include <iostream>

int main()
{
  int number = 1024;
  int *pointer = &number;

  std::cout << "pointer value:" << *pointer << std::endl;
}
```

**WARNING** It is illegal to assign an `int` value to a pointer, even if the variable's value happens to be 0.


[Return to the main article](/techtalk/c++)
