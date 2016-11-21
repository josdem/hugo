+++
categories = ["techtalk", "code"]
date = "2016-11-21T08:18:18-06:00"
tags = ["josdem", "techtalks", "programming", "technology","machine learning", "matrices operations", "octave"]
title = "Linear Algebra"

+++

Linear algebra is the branch of mathematics concerning vector spaces and linear mappings between such spaces. It includes the study of lines, planes, and subspaces, but is also concerned with properties common to all vector spaces.

**Matrices and Vectors Multiplication**

I would like to start talking about how to multiply together two matrices. We will start with a special case of that, of matrix vector multiplication. Let's consider the following example:

`$A = \begin{bmatrix}
        1 & 3 \\
        4 & 0 \\
        2 & 1 \\
\end{bmatrix}$`

`$b = \begin{bmatrix}
   1 \\
   5 \\
\end{bmatrix}$`

So we want to multiply a 3x2 Matrix with a 2x1 vector, the result of this would be a 3x1 vector. The multiplication would be as follow:

First row from matrix and column of the vector:

`$1 x 1 + 3 x 5 = 16 $`

Second row from matrix and column of the vector:

`$4 x 1 + 0 x 5 = 4 $`

Third row from matrix and column of the vector:

`$2 x 1 + 1 x 5 = 7 $`

And the result is:

`$c = \begin{bmatrix}
   16 \\
   4 \\
   7 \\
\end{bmatrix}$`

You can use some great application opensource to solve this problem such as Octave: [Octave](https://www.gnu.org/software/octave/)

Using the command window in Octave you can create a matrix manually like this:

```bash
A = [1,3;4,0;2,1]
```

and a vector like this:

```bash
b = [1;5]
```

Note: By convention, matrices are represented by uppercase letter and vectors with lowercase

So we have that if we type this command in Octave:

```bash
>> A * b
ans =
   16
    4
    7
```

[Return to the main article](/techtalk/machine_learning/machine_learning)
