+++
categories = ["c++", "cplusplus", "g++"]
date = "2016-07-03T11:33:29-05:00"
tags = ["josdem", "techtalks", "c++", "istringstream"]
title = "The IO Library: istringstream"

+++

An istringstream is often used when we have some work to do on an entire line, and other work to do with individual words within a line.

As example, assume we have a file that list people and their associated phone numbers. Some people have only one number, but others have several, our ouptut file might look like the following:

```
josdem 5516827055
eric 55123456789 5500011122
mario 7771234567 7770987654 7770011223
```

Let's define a class to represent our data:

```c++
struct Person{
  strig name;
  vector<string> phones;
}
```

Let's create a program that reads a data file and build up a vector of person. Each element in the vector will correspond to one record in the file.

```c++
#include <iostream>
#include <sstream>
#include <fstream>
#include <vector>
using namespace std;

struct Person
{
  string name;
  vector<string> phones;
};

void print_person(vector<Person> people){
  for(Person p: people){
    cout << p.name << endl;
  }
}

int main()
{
  string line, word;
  vector<Person> people;
  ifstream file("persons.txt");
  while(getline(file, line))    // 1
  {
    Person person;
    istringstream record(line); // 2
    record >> person.name;      // 3
    while(record >> word)       // 4
    {
      person.phones.push_back(word);
    }
    people.push_back(person);
  }

  print_person(people);         // 5
}
```
1. `getline` read an entire record from the file
2. Bind record to the line we just read
3. Read the name
4. Read the phone
5. Prints person's name

An `ostringstream` is useful when we need to build up our output a little at  time but do not want to print the output until later. For example, we might want to validate and reformat the phone numbers we read in the previous code, after formatted we want to print a new file containing the reformatted numbers.

```c++
#include <iostream>
#include <sstream>
#include <fstream>
#include <vector>
using namespace std;

struct Person
{
  string name;
  vector<string> phones;
};

void format(vector<Person> people){
  for(Person &entry: people){        // 1
    ostringstream formatted;
    formatted << entry.name;
    for(string &nums: entry.phones)  // 2
    {
      formatted << nums;
    }
    cout << formatted.str() << endl; // 3
  }
}

int main()
{
  string line, word;
  vector<Person> people;
  ifstream file("persons.txt");
  while(getline(file, line))
  {
    Person person;
    istringstream record(line);
    record >> person.name;
    while(record >> word)
    {
      person.phones.push_back(word);
    }
    people.push_back(person);
  }

  format(people);
}
```

1. For each entry in people
2. For each number
3. Print the name and the formatted phones

[Return to the main article](/techtalk/c++)
