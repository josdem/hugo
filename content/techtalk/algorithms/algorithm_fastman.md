+++
date = "2016-05-10T20:52:26-05:00"
draft = true
title = "Fastman"

+++

Write a program that breaks up a string of words with no spaces into a string with appropiate spaces.

* Consider an example that's rich enough but no tedious:
 * "fastman" --> "fast man"
* Consider an empty string or null
* The words come from custom dictionary

**Solution**

```groovy
def dictionary = ["fast","fat","man","woman"]
String string = "fastman"
String result = ""

if(string?.length() > 0){
  for(i=0;i<string.length()-1;i++){
    for(j=string.length();j!=i;j--){
     if(dictionary.contains(string.substring(i,j))){
       result += "${string.substring(i,j)} "
     }
    }
  }
}

assert 'fast man' == result.trim()
```

[Return to the main article](/techtalk/algorithms)
