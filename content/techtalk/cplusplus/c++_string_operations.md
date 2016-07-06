+++
categories = ["techtalk", "code"]
date = "2016-07-06T16:28:09-05:00"
tags = ["josdem", "techtalks", "programming", "technology","c++","cplusplus", "strings operations"]
title = "String Operations"

+++

There are some additional ways to construct strings

```c++
#include <iostream>
#include <assert.h>

using namespace std;

int main()
{
  const char *cp = "josdem";

  string s1=cp;         // <1>
  string s2(cp);
  assert(cp == s1);
  assert(cp == s2);

  string s3(cp+3,3);    // <2>
  assert(s3 == "dem");

  string s4(cp,2,2);
  assert(s4 == "sd");   // <3>

  string s5(cp,3);
  assert(s5 == "jos");  // <4>

  return 0;
}
```

1. Copies the string
2. Copy 3 characters starting at index 3
3. Copy 2 characters starting at index 2
4. Copy 3 characters starting from the beginning

[Return to the main article](/techtalk/c++)
