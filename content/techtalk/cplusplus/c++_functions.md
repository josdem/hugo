+++
date = "2016-06-18T10:38:11-05:00"
title = "Functions"
categories = ["techtalk","code","c++"]
tags = ["josdem","techtalks","programming","technology","c++"]
description = "A function is a group of statements that together perform a task. Every C++ program has at least one function, which is main(), and all the most trivial programs can define additional functions."
+++

As an example, we'll write a function to determine the factorial of a given number. 5 Factorial is: 1 * 2 * 3 * 4 * 5 = 120

```c++
#include <iostream>
using namespace std;

int factorial(unsigned int value){
  unsigned int result = 1;
  while(value > 1){
    result *= value--;
  }
  return result;
}

int main(){
  cout << "Factorial 5 is: " << factorial(5) << endl;
  return 0;
}
```

In C++, variables have **scope** and objects have **lifetimes**. It is important to understand both of there concepts.

* The scope of a name is *the part of the program's text* in which that name is visible.
* The lifetime of an object is *the time during the program's execution* that the object exists.

Another important aspect about functions are arguments, when a parameter is a reference, we say that its corresponding argument is **passed by reference** a reference parameter is an alias for the object to which it is bound; that is, the parameter is an alias for its corresponding argument.

When the argument value is copied, the parameter and argument are independent objects. We say such argument are **passed by value**.

**Pointer Parameter**

```c++
#include <iostream>
using namespace std;

void reset(unsigned int *p){
  cout << "reseting value" << endl;
  *p = 0;
}

int main(){
  unsigned int i = 40;
  cout << "address i: " << &i << endl;
  cout << "value i: " << i << endl;
  reset(&i);
  cout << "value i: " << i << endl;
  cout << "address i: " << &i << endl;
}
```

output

```
address i: 0x7ffe760a9944
value i: 40
reseting value
value i: 0
address i: 0x7ffe760a9944
```

**By reference**

```c++
#include <iostream>
using namespace std;

void reset(unsigned int &reference){
  cout << "reseting value" << endl;
  reference = 0;
}

int main(){
  unsigned int i = 40;
  cout << "address i: " << &i << endl;
  cout << "value i: " << i << endl;
  reset(i);
  cout << "value i: " << i << endl;
  cout << "address i: " << &i << endl;
}
```

output

```
address i: 0x7ffcf264e074
value i: 40
reseting value
value i: 0
address i: 0x7ffcf264e074
```

**By Value**

```c++
#include <iostream>
using namespace std;

void reset(unsigned int parameter){
  cout << "changing value" << endl;
  parameter = 0;
}

int main(){
  unsigned int i = 40;
  cout << "address i: " << &i << endl;
  cout << "value i: " << i << endl;
  reset(i);
  cout << "value i: " << i << endl;
  cout << "address i: " << &i << endl;
}
```

output

```
address i: 0x7ffd56db2444
value i: 40
reseting value
value i: 40
address i: 0x7ffd56db2444
```

## Reference return

Whether a function call is an lvalue depends on the return type of the function. Calls to functions that return references are lvalues.

```c++
#include <iostream>
using namespace std;

char &get_val(string &s, unsigned int index)
{
  return s[index];
}

int main()
{
  string s = "this is a value";
  cout << s << endl;
  get_val(s,0) = 'T';
  cout << s << endl;
  return 0;
}
```

The return value is a reference.

Output

```
this is a value
This is a value
```

## Default Arguments

Some functions have parameters that are given a particular value in most, but not all calls. For example if we create a window we might want a particular height, width and title, also we might want to allow users to pass values other than defaults.

```c++
#include <gtkmm.h>
using namespace std;

void screen(unsigned int width = 640, unsigned int height = 480, string title = "GTK Window"){
  Gtk::Window window;
  window.set_default_size(width, height);
  window.set_title(title);

  Gtk::Main::run(window);
}

int main (int argc, char *argv[])
{
  Gtk::Main kit(argc, argv);
  screen();
  return 0;
}
```

If we want to use the default argument, we omit that argument when we call the function.

```c++
screen();     // Equivalent to screen(640, 480, "GTK Window")
screen(100);  // Equivalent to screen(100, 480, "GTK Window")
screen(100,200, "My Window"); // Different values than default
```

Tu run this example:

```g++ window.cpp -o window.o `pkg-config --libs --cflags gtkmm-2.4````

[Return to the main article](/techtalk/c++)
