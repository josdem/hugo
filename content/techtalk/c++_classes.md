+++
date = "2016-06-19T13:06:03-05:00"
draft = true
title = "Classes"

+++

Classes provides great level of data abstraction and provides two important advantages:

* Class internals are protected from inadvertent user-level errors, which might corrupt the state of the object.
* The class implementation may evolve over time in response to changing requirements or bug reports without requiring change in user-level code.

Any C++ program where you implement a class with public and private members is an example of data abstraction. Consider the following example:

```c++
#include <iostream>
using namespace std;

class Adder{
  public:
    void addNum(int number)
    {
      total += number;
    }
    int getTotal()
    {
      return total;
    };
  private:
    int total = 0;
};
int main( )
{
  Adder adder;

  adder.addNum(10);
  adder.addNum(20);
  adder.addNum(30);

  cout << "Total " << adder.getTotal() << endl;
  return 0;
}
```

The public members addNum and getTotal are methods to the outside world and a user needs to know them to use the class. The private member total is something that the user doesn't need to know about, but is needed for the class to operate properly.

## Constructors

A class constructor is a special member function of a class that is executed whenever we create new objects of that class.
A constructor will have exact same name as the class and it does not have any return type. Constructors is used for setting initial values for certain member variables.

```c++
#include <iostream>
using namespace std;

class Rectangle
{
  public:
    Rectangle(unsigned int w, unsigned int h)
    {
      width = w;
      height = h;
    }
    double getArea()
    {
      return width * height;
    }

  private:
    double width;
    double height;
};

int main()
{
  Rectangle rectangle(10,5);
  cout << "Area is: " << rectangle.getArea() << endl;
  return 0;
}
```

## Destructors

Destructor is used for releasing resources before coming out of the program like closing files, releasing memories etc.

```c++
#include <iostream>
using namespace std;

class Rectangle
{
  public:
    Rectangle(unsigned int w, unsigned int h)
    {
      width = w;
      height = h;
    }
    ~Rectangle()
    {
      cout << "Deleting rectangle" << endl;
    }
    double getArea()
    {
      return width * height;
    }

  private:
    double width;
    double height;
};

int main()
{
  Rectangle rectangle(10,5);
  cout << "Area is: " << rectangle.getArea() << endl;
  return 0;
}
```

Output:

```
Area is: 50
Deleting rectangle
```

## Using this

Every object in C++ has access to its own address through an important pointer called this pointer. The this pointer is an implicit parameter to all member functions.


```c++
#include <iostream>
using namespace std;

class Rectangle
{
  public:
    Rectangle(unsigned int w, unsigned int h)
    {
      width = w;
      height = h;
    }

    double getArea()
    {
      return width * height;
    }

    int compare(Rectangle rectangle)
    {
      return this->getArea() > rectangle.getArea();
    }

  private:
    double width;
    double height;
};

int main()
{
  Rectangle rec1(10,6);
  Rectangle rec2(10,5);

  if(rec1.compare(rec2))
  {
    cout << "Rectangle 2 is smaller than Rectangle 1" <<endl;
  }
  else
  {
    cout << "Rectangle 2 is equal to or larger than Rectangle 1" <<endl;
  }
  return 0;
}
```

Output:

```
Rectangle 2 is smaller than Rectangle 1
```

##  To return reference to the calling object

```c++
#include <iostream>
using namespace std;

class Rectangle
{
  public:
    Rectangle(unsigned int w, unsigned int h)
    {
      width = w;
      height = h;
    }

    double getArea()
    {
      return width * height;
    }

    Rectangle& reference()
    {
      return *this;
    }

  private:
    double width;
    double height;
};
```

[Return to the main article](/techtalk/c++)
