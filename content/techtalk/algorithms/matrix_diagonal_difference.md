+++
date = "2018-01-31T20:52:26-05:00"
tags = ["josdem", "techtalks","programming","technology","algorithms"]
categories = ["techtalk", "code"]
title = "Matrix Diagonal Difference"
description = "Given a square matrix of size N x N, calculate the absolute difference between the sums of its diagonals."
+++

**Input**

Given a square matrix of size `$N x N$`, calculate the absolute difference between the sums of its diagonals.

**Output**

Print the absolute difference between the two sums of the matrix's diagonals as a single integer.

**Sample Input**

`$matrix = \begin{bmatrix}
        11 & 2 & 4 \\
         4 & 5 & 6 \\
        10 & 8 &-12 \\
\end{bmatrix}$`

**Sample Output**

`$15$`

**Explanation**

The primary diagonal is:

`$\begin{bmatrix}
   11 \\
    5  \\
   -12 \\
\end{bmatrix}$`

Sum across the primary diagonal: `$11 + 5 - 12 = 4$`

The secondary diagonal is:

`$\begin{bmatrix}
    4 \\
    5 \\
   10 \\
\end{bmatrix}$`

Sum across the secondary diagonal: `$4 + 5 + 10 = 19$`

Difference: `$|4 - 19| = 15$`

Note: `$|x|$` is [absolute value](https://en.wikipedia.org/wiki/Absolute_value) function

**Solution**

```java
import java.lang.Math;

public class MatrixDiagonalSubstractor {

  private int sum(int[][] matrix){
    int diagonalA = 0;
    int diagonalB = 0;
    int n = matrix.length;

    for(int i=0; i<n; i++){
      for(int j=0; j<n; j++){
        if(i==j){
          diagonalA = diagonalA + matrix[i][j];
        }
        if(j==n-i-1){
          diagonalB = diagonalB + matrix[i][j];
        }
      }
    }
    return Math.abs(diagonalA - diagonalB);
  }

  public static void main(String[] args){
    int[][] matrix = {
      {11, 2,  4},
      { 4, 5,  6},
      {10, 8,-12},
    };

    int result = new MatrixDiagonalSubstractor().sum(matrix);
    assert 15 == result;
  }

}
```

This kind of solution has `$O(N^2)$` complexity, since it's performance is directly proportional to the square of the size of the input data set. This is common with algorithms that involve nested iterations over the data set. For more information about it, please refer [A Beginner's Guide to Big O nottation](https://rob-bell.net/2009/06/a-beginners-guide-to-big-o-notation/)

To download the code:

```bash
git clone https://github.com/josdem/algorithms-workshop.git
cd matrix-diagonal-difference
```

To run Java code:

```bash
javac MatrixDiagonalSubstractor.java
java -ea MatrixDiagonalSubstractor
```


[Return to the main article](/techtalk/algorithms)
