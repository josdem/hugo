+++
date = "2016-07-01T17:45:34-05:00"
draft = true
categories = ["c++", "cplusplus", "g++"]
tags = ["josdem", "techtalks", "c++", "fstream", "ofstream", "ifstream"]
title = "The IO Library: fstream"
description = "C++ provides important classes to perform output and input of characters to/from files"
+++

C++ provides the following classes to perform output and input of characters to/from files:

* `ofstream`: Stream class to write on files
* `ifstream`: Stream class to read from files
* `fstream`: Stream class to both read and write from/to files.

## Writing text into a file

```c++
#include <iostream>
#include <fstream>
using namespace std;

int main()
{
  ofstream file;
  file.open("some_file.txt");
  file << "This is a new line \n";
  file.close();
  return 0;
}
```

## Reading from a file

```c++
#include <iostream>
#include <fstream>
using namespace std;

int main()
{
  string line;
  ifstream file("some_file.txt");
  if(file.is_open())
  {
    while(getline(file,line))
    {
      cout << line << endl;
    }
    file.close();
  }
  else cout << "Unable to open file" << endl;
  return 0;
}
```

## Copying a file

```c++
#include <iostream>
#include <fstream>
using namespace std;

int main()
{
  ifstream infile("some_file.txt");
  ofstream outfile("copied_file.txt");
  outfile << infile.rdbuf();
  infile.close();
  outfile.close();
  return 0;
}
```

The C++ fstreams are buffered internally. They use an efficient buffer size. `rdbuf()` just returns a pointer to the buffer, so just copy one stream buffer to a stream and the internal magic will do an efficient copy of one stream to the other.


[Return to the main article](/techtalk/c++)
