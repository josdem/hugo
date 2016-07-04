+++
date = "2016-06-17T18:11:00-05:00"
draft = true
title = "Exception Handling"

+++

Dealing with unexpected behaviour can be one of the most difficult parts in a system, even though there is so common, so we need a way to detect such behaviour as exceptions and handle it in a proper manner.

```c++
#include <iostream>
using namespace std;

void throwAnException(){
  throw runtime_error("Runtime exception");
}

int main(){
  try {
    throwAnException();
  } catch(runtime_error &rte){
    cout << rte.what() << endl;
  }
}
```

The type `runtime_error` is one of the standard library exception types. When what() is called a const char pointer is returned that points at a string that was passed into the constructor, in this case `Runtime Exception`

## Standard Exception

The C++ Standard library provides a base class specifically designed to declare objects to be thrown as exceptions. It is called `std::exception`. This class has a virtual member function called what that returns a null-terminated character sequence (of type char \*) and that can be overwritten in derived classes to contain some sort of description of the exception.

```c++
#include <iostream>
using namespace std;

class BusinessException : public exception {

  public: virtual const char* what() const throw()
  {
    return "Business Exception";
  }

};

void throwAnException(){
  throw BusinessException();
}

int main(){

  try {
    throwAnException();
  } catch(BusinessException &rte){
    cout << rte.what() << endl;
  }

}
```

[Return to the main article](/techtalk/c++)

