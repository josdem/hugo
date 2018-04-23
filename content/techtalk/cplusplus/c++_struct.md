+++
date = "2016-06-15T20:56:52-05:00"
title = "Data Structures"
categories = ["techtalk", "code","c++"]
tags = ["josdem", "techtalks", "programming", "technology","c++","cplusplus", "Data Structures"]
description = "A data structure is a way to group together related data elements and a strategy for using those data."
+++

A data structure is a way to group together related data elements and a strategy for using those data.

```c++
#include <iostream>

struct Item {
  std::string name;
  unsigned cuantity;
  double price;
};

int main(){
  Item item;
}
```

Here we are populating our struct data and simplifying out std input and outputs

```c++
#include <iostream>
using namespace std;

struct Item {
  string name;
  unsigned quantity;
  double price;
};

int main(){
  Item item;

  cout << "Item name: " << endl;
  cin >> item.name;
  cout << "Quantity: " << endl;
  cin >> item.quantity;
  cout << "Price: " << endl;
  cin >> item.price;
  cout << "Total: " << item.quantity * item.price << endl;

  return 0;
}
```

[Return to the main article](/techtalk/c++)
