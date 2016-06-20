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

[Return to the main article](/techtalk/c++)
