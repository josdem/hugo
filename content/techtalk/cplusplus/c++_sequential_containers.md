+++
categories = ["techtalk", "code"]
date = "2016-07-06T09:38:00-05:00"
tags = ["josdem", "techtalks", "programming", "technology","c++","cplusplus"]
title = "Sequential Containers"

+++

A *container* holds a collection of objects of a specified type. The **sequential containers** let the programmer control the order in which elements are stored and accessed. With the exception of *array*, which is a fixed-size container, the containers provide efficient, flexible memory management. Ordinarily, is best to use vector unless there is a good reason to prefer another container.

## Copy a vector

To create a container as a copy of another container, the container and elements type must match.

```c++
#include <iostream>
#include <vector>

using namespace std;

int main()
{
  vector<string> names = {"josdem","eric","mario"};
  vector<string> copy(names);

  for(string &item: copy)
  {
    cout << item << endl;
  }
  return 0;
}
```

We can use assignment operation which replace the elements in copy with copies of the elements in names and must be the same type.

```c++
#include <iostream>
#include <vector>

using namespace std;

int main()
{
  vector<string> names = {"josdem","eric","mario"};
  vector<string> copy;

  copy = names;

  for(string &item: copy)
  {
    cout << item << endl;
  }
  return 0;
}
```

We can also initialize the sequential container from a size and an optional element initializer.

```c++
#include <iostream>
#include <vector>

using namespace std;

int main()
{
  vector<string> names(10,"josdem");

  for(string &item: names)
  {
    cout << item << endl;
  }
  return 0;
}
```

## Relational Operations

Every container type supports the equality operators (== and !=) also support the relational operators (>, >=, <, <=). The right and left hand operands must be the same kind of container and must hold elemtns of the same type.

* If both containers are the same size and all the elements are equal, then the two containers are equal; otherwise, they are unequal.
* If the containers have different sizes but every element of the smaller one is equal to the corresponfing element of the larger one, the the smaller is less than the other.
* If neither container is an initial subsequence of the other, then the comparison depends on comparing the first unequal elements.

```c++
#include <iostream>
#include <vector>
#include <assert.h>

using namespace std;

int main()
{
  vector<int> v1 = {1,2,3,5,8,13,21,34};
  vector<int> v2 = {1,3,9};
  vector<int> v3 = {1,2,3,5};
  vector<int> v4 = {1,2,3,5,8,13,21,34};

  assert(v1 < v2);   // <1>
  assert(v1 < v3);   // <2>
  assert(v1 == v4);  // <3>
  assert(v1 == v2);  // <4>

  return 0;
}
```

1. true; v1 and v2 differ at element[1]; v1 is less than v2
2. false; all elements are equal, but v3 has fewer of them
3. true; each element is equal and v1 and v4 have the same size
4. false; v2 has fewer element than v1

## Sequential Container Operations

Containers provide flexible memory management. We can add or remove elements dynamically changing the size of the container at run time.

```c++
#include <iostream>
#include <vector>
#include <assert.h>

using namespace std;

int main()
{
  vector<int> v = {1,2,3,4,5};
  vector<int>::iterator p = v.begin();

  v.insert(p+2,8);

  for(int &item: v)
  {
    cout << item << endl;
  }

  return 0;
}
```

output:

```
1
2
8
3
4
5
```

Creates an element with value 8 before the element denoted by iterator p.

## Erasing Elements

There are several ways to remove elements.

```c++
#include <iostream>
#include <vector>
#include <assert.h>

using namespace std;

int main()
{
  vector<int> v = {1,2,3,4,5};
  vector<int>::iterator p = v.begin();

  v.erase(p+2);

  for(int &item: v)
  {
    cout << item << endl;
  }

  return 0;
}
```

Removes the element denoted by the iterator p.

---
| function        | Description                                                 |
|---------------  |------------------------------------------------------------ |
| v.pop_back()    | Removes last element                                        |
| v.pop_front()   | Removes first element                                       |
| v.erase(b,e)    | Removes the range of elements denoted by iterators b and e  |
| v.clear()       | Removes all elements                                        |
---

**WARNING:** The programmer must ensure that element(s) exist before removing them.


[Return to the main article](/techtalk/c++)
