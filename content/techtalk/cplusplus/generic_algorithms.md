+++
categories = ["techtalk", "code"]
date = "2016-07-08T13:30:03-05:00"
tags = ["josdem", "techtalks", "programming", "technology","cplusplus","c++ algorithms"]
title = "Generic Algorithms"
description = "The library provides a set of algorithms, most of which are independent of any particular container type. Those algorithms implement common functions such as sorting and searching. Most of the algorithms are defined in the algorithm header. In general, the algorithms do not work directly on a container. Instead, they operate by traversing a range of elements bounded by two operators."
+++

The library provides a set of algorithms, most of which are independent of any particular container type. Those algorithms implement common functions such as sorting and searching. Most of the algorithms are defined in the algorithm header. In general, the algorithms do not work directly on a container. Instead, they operate by traversing a range of elements bounded by two operators.

For example: Lest's suppose we have a `vector` of `int` and we want to know if that `vector` holds a particular value.

```c++
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int main()
{
  int value=36;

  vector<int> numbers = {2,12,18,36,40,58};
  auto result = find(numbers.cbegin(), numbers.cend() ,value);
  cout << (result == numbers.cend() ? "is not present" : "found!") << endl;

}
```

Because `find` operates in terms of iterators, we can use the same `find` function to look for values in any type of container. For example, because pointers act like iterators, we can use `find` in an array.

```c++
#include <iostream>
#include <assert.h>
#include <algorithm>

using namespace std;

int main()
{
  int value=36;

  int numbers[] = {2,12,18,36,40,58};
  auto result = find(begin(numbers), end(numbers) ,value);
  assert(*result == 36);

}
```

## Read-Only algorithms

A numner of the algorithms read, but never write to, one of those is `accumulate`, which is defined in the `numeric` header.

```c++
#include <iostream>
#include <vector>
#include <assert.h>
#include <algorithm>

using namespace std;

int main()
{

  vector<int> numbers = {2,12,18,36,40,58};
  int sum = accumulate(numbers.cbegin(), numbers.cend(), 0);
  assert(sum == 166);

}
```

sets `sum` equal to the sum of the elements in the vector, using 0 as starting point of summation.

## Algorithms that write container elements

Some algorithms assign new values to the elements in a sequence.

```c++
#include <iostream>
#include <vector>
#include <algorithm>

using namespace std;

int main()
{

  vector<int> numbers = {2,12,18,36,40,58};
  fill(numbers.begin(), numbers.begin() + numbers.size()/2, 0);

  for(int it: numbers){
    cout << it << endl;
  }

}
```

Output:

```
0
0
0
36
40
58
```

[Return to the main article](/techtalk/c++)
