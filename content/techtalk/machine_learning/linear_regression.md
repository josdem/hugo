+++
categories = ["techtalk", "code"]
date = "2016-11-13T17:15:45-06:00"
tags = ["josdem", "techtalks", "programming", "technology"]
title = "Linear Regression"

+++

## Model Representation

In supervised learning, we are given a data set and already know what our correct output should look like, having the idea that there is a relationship between the input and the output.

Linear regression with one variable is also known as "univariate linear regression."
Univariate linear regression is used when you want to predict a single output value  from a single input value . We're doing supervised learning here, so that means we already have an idea about what the input/output cause and effect should be.

As a result of an experiment, four (x,y) data points were obtained, (1,6), (2,5), (3,7), and (4,10) marked in red cross marker

<img src="/img/techtalks/machine_learning/linear_regression.png">

The solution is to find a line

```math
y = b1 + b2x
```

that best fits these four points. In other words, we would like to find the numbers b1 and b2 that approximately solve the overdetermined linear system, so we have that:

```math
b1 + 1b2 = 6
b1 + 2b2 = 5
b1 + 3b2 = 7
b1 + 4b2 = 10
```

The "error", at each point, between the line fit and the data is the difference between the right- and left-hand sides of the equations above. The least squares approach to solving this problem is to try to make the sum of the squares of these errors as small as possible; that is, to find the minimum of the function:



