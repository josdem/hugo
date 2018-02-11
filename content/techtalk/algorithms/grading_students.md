+++
date = "2018-02-10T20:52:26-05:00"
tags = ["josdem", "techtalks","programming","technology"]
categories = ["techtalk", "code","grading students", "coding challenge", "algorithms"]
title = "Grading Students"
+++

At HackerLand University, a passing grade is any grade `40` points or higher on a `100` point scale. Sam is a professor at the university and likes to round each student’s grade according to the following rules:

* If the difference between the grade and the next higher multiple of `5` is less than `3`, round to the next higher multiple of `5`
* If the grade is less than `38`, don’t bother as it’s still a failing grade

Automate the rounding process then round a list of grades and print the results.

**Constraints**

* `$1 \le n \le 60$`
* `$0 \le gradei \le 100$`

**Sample Input**

```bash
[73, 67, 38, 33]
```

**Sample Output**

```bash
[75, 67, 40, 33]
```

**Explanation**

The first grade, `73` is two below the next higher multiple of `5`, so it rounds to `75`.
`67` is `3`  points less than the next higher multiple of `5` so it doesn’t round.
`38`, like `73`, rounds up to next higher multiple of `5`, or `40` in this case.
`33` is less than `38`, so it does not round.

**Solution Java 7**

```java
import java.util.List;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.stream.Collectors;

public class StudentsGradeCalculator {

  private List<Integer> compute(List<Integer> grades){
    List<Integer> result = new ArrayList<Integer>();
    for(Integer grade: grades){
      if(grade >= 38 && grade % 5 >= 3){
        grade = grade + 5 - grade % 5;
      }
      result.add(grade);
    }
    return result;
  }

  public static void main(String[] args){
    List<Integer> grades = Arrays.asList(73, 67, 38, 33);
    List<Integer> result = new StudentsGradeCalculator().compute(grades);
    assert 4 == result.size();
    assert 75 == result.get(0);
    assert 67 == result.get(1);
    assert 40 == result.get(2);
    assert 33 == result.get(3);
  }

}
```

**Explanation**

Observe what happens when we increment grades by one and then we get modulus by 5.

```bash
38 % 5 = 3
39 % 5 = 4
40 % 5 = 0
41 % 5 = 1
42 % 5 = 2
43 % 5 = 3
44 % 5 = 4
45 % 5 = 0
46 % 5 = 1
47 % 5 = 2
48 % 5 = 3
49 % 5 = 4
50 % 5 = 0
```

So that is why we are verifying `grade % 5 >= 3` in our if conditional

This solution kind is `O(N)` since the algorithm performance will grow linearly and in direct proportion to the size of the input data set.

Let's consider the Java 8 version using lambda

```java
import java.util.List;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.stream.Collectors;

public class StudentsGradeCalculator {

  private List<Integer> compute(List<Integer> grades){
    return grades
      .stream()
      .map(it -> (it >=38 && it % 5 >=3) ? it + 5 - it % 5 : it)
      .collect(Collectors.toList());
  }

  public static void main(String[] args){
    List<Integer> grades = Arrays.asList(73, 67, 38, 33);
    List<Integer> result = new StudentsGradeCalculator().compute(grades);
    assert 4 == result.size();
    assert 75 == result.get(0);
    assert 67 == result.get(1);
    assert 40 == result.get(2);
    assert 33 == result.get(3);
  }

}
```

To Download the code:


```bash
git clone https://github.com/josdem/algorithms-workshop.git
cd grading-students
```

To run the code:

```bash
javac StudentsGradeCalculator.java
java -ea StudentsGradeCalculator
```


[Return to the main article](/techtalk/algorithms)
