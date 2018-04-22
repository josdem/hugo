+++
title = "Is Pangram"
categories = ["techtalk","code"]
tags = ["josdem","techtalks","programming","technology","algorithms","java"]
date = "2017-11-29T15:42:15-06:00"
description = "Given a sentence, check whether it is a pangram or not."
+++


Given a sentence, check whether it is a [pangram](https://en.wikipedia.org/wiki/Pangram) or not.

**Explanation**

For sentence = "The quick brown fox jumps over the lazy dog.", the output should be

```
isPangram(sentence) = true.
```

For sentence = "Hello World!", the output should be

```
isPangram(sentence) = false.
```


**Solution**

```java
public class PangramVerifier {

  private static final int ASCII_ALPHABET_SUM = 2015;

  private Boolean isPangram(String quote){
    Integer sum = quote.toUpperCase().chars().filter(x -> x > 64 & x < 91).distinct().sum();
    return sum == ASCII_ALPHABET_SUM;
  }

  public static void main(String[] args){
    String quote = "The quick brown fox jumps over the lazy dog";
    Boolean result = new PangramVerifier().isPangram(quote);
    assert result;
  }

}
```


To download the code:

```bash
git clone https://github.com/josdem/algorithms-workshop.git
cd is-pangram
```

To run the code:

```bash
javac PangramVerifier.java
java -ea PangramVerifier
```


[Return to the main article](/techtalk/algorithms)
