+++
date = "2016-06-16T17:53:14-05:00"
title = "Loops and Collections"
categories = ["c++", "cplusplus", "g++"]
tags = ["josdem", "techtalks", "c++", "fstream", "ofstream", "ifstream"]
description = "If we want to change the value of the character in a string, we must define the loop variable as a reference type."
+++

If we want to change the value of the character in a string, we must define the loop variable as a reference type.

```c++
#include <iostream>
using namespace std;

int main(){
  string nickname = "josdem";

  for(char &c: nickname){
    c = toupper(c);
  }

  cout << nickname << endl;
}
```

## Vector

A vector is a collection of objects, all which have the same type. Every object in the collection has an associated index, which gives access to that object. To use a vector, we must include the header.

```c++
#include <iostream>
#include <vector>

using namespace std;

int main(){
  vector<int> integers = {1,2,3,4,5};

  for(int i=6; i < 10; i++){
    integers.push_back(i);
  }

  for(int i: integers){
    cout << i << endl;
  }

}
```

**Vector operations**

* `vector.empty();`
* `vector.size();`
* `vector.push_back(t)`
* `vector[n];`

## Arrays

An array is a data structure that is similar to the vector, but it has a fixed size; so it offer a better run time performance.

```c++
#include <iostream>

using namespace std;

int main(){
  int integers[10] = {0,1,2,3,4};

  for(int i=5; i < 10; i++){
    integers[i] = i;
  }

  for(int i: integers){
    cout << i << endl;
  }

}
```

**WARNING** We cannot initialize an array as a copy of anorher array.

You can define multidimensional arrays `int matrix[3][4]` this is an array of size 3; each element is an array of ints of size 4

[Return to the main article](/techtalk/c++)


