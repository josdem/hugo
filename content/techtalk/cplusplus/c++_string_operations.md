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

## The substr operation

The `substr` operation returns a `string` that is a copy of part or all of the original string. We can pass `substr` an optional starting position and count.

```c++
#include <iostream>
#include <assert.h>

using namespace std;

int main()
{
  string s("josdem");
  string s1 = s.substr(0,3);
  string s2 = s.substr(3);
  string s3 = s.substr(3,5);

  assert(s1=="jos");
  assert(s2=="dem");
  assert(s3=="dem");

  try{
    string s4 = s.substr(12); // <1>
  }catch(const out_of_range ex){
    cout << "Out of Range error: " << ex.what() << endl;
  }

  return 0;
}
```

1. Throws an out_of_range exception since exceeds the size of the string

Output:

```
Out of Range error: basic_string::substr: __pos (which is 12) > this->size() (which is 6)
```

The string type supports the sequential container assignment operations and the `replace`, `insert` and `erase` operations.

```c++
#include <iostream>
#include <assert.h>

using namespace std;

int main()
{
  string s("josdem");

  s.insert(s.size(), "!");
  assert(s == "josdem!");

  s.erase(s.size()-1 , 1);
  assert(s == "josdem");

  s.replace(3,5,"e");
  assert(s == "jose");

  return 0;
}
```

## String search operations

The string class provides six different search functions. Each operations returns a `size_type` value that is the index of where the match occurred. It there is no matchm the function returs a `static` member named `npos`. The library defines `npos` as `const size_type` initialized by -1.

```c++
#include <iostream>
#include <assert.h>

using namespace std;

int main()
{
  string s("josdem is a software developer, is a DJ");

  string::size_type pos = s.find("software");
  assert(pos == 12);

  pos = s.rfind("is");
  assert(pos == 32);

  pos = s.find_first_of('i');
  assert(pos == 7);

  pos = s.find_last_of('i');
  assert(pos == 32);

  cout << pos << endl;

  return 0;
}
```

## Numeric conversions

The new standard introduced several functions that convert between numeric data and library `strings`

```c++
#include <iostream>
#include <assert.h>

using namespace std;

int main()
{

  int i = 36;
  string s = to_string(i);
  assert(s == "36");

  double d = stod(s); // <1>
  assert(d == 36);

  string number = "7055 is my number";
  int n = stoi(number); // <2>
  assert(7055 == n);

  number = "5516827055 is my number";
  long l = stol(number); // <3>
  assert(5516827055 == l);

  return 0;
}
```

1. String to double
2. String to int
3. String to long

[Return to the main article](/techtalk/c++)
