+++
date = "2016-06-14T17:35:25-05:00"
draft = true
title = "C++ Basics"

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

[Return to the main article](/techtalk/c++)
