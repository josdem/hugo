+++
title = "Linear Regression"
categories = ["techtalk", "code","java"]
tags = ["josdem", "techtalks", "programming", "technology","java"]
date = "2016-11-13T17:15:45-06:00"
description = "In supervised learning, we are given a data set and already know what our correct output should look like, having the idea that there is a relationship between the input and the output."
+++

## Model Representation

In supervised learning, we are given a data set and already know what our correct output should look like, having the idea that there is a relationship between the input and the output.

Linear regression with one variable is also known as "univariate linear regression."
Univariate linear regression is used when you want to predict a single output value  from a single input value . We're doing supervised learning here, so that means we already have an idea about what the input/output cause and effect should be.

As a result of an experiment, four (x,y) data points were obtained, (1,6), (2,5), (3,7), and (4,10) marked in red cross marker

<img src="/img/techtalks/machine_learning/linear_regression.png">

The solution is to find a line

$\theta = \beta 1 + \beta 2 x$

that best fits these four points. In other words, we would like to find the numbers `$\beta 1$` and `$\beta 2$` that approximately solve the overdetermined linear system, so we have that:

`$\beta 1 + 1 \beta 2 = 6 $`

`$\beta 1 + 2 \beta 2 = 5 $`

`$ \beta 1 + 3 \beta 2 = 7 $`

`$ \beta 1 + 4 \beta 2 = 10 $`


The "error", at each point, between the line fit and the data is the difference between the right- and left-hand sides of the equations above. The least squares approach to solving this problem is to try to make the sum of the squares of these errors as small as possible; that is, to find the minimum of the function:

$$\sum_{i=1}^m [y - \theta x]^2$$

So we have that:

$$S(\beta 1, \beta 2) = [6 - (\beta 1 + 1 \beta 2)]^2 + [5 - (\beta 1 + 2 \beta 2)]^2 + [7 - (\beta 1 + 3 \beta 2)]^2+ [10 - (\beta 1 + 4 \beta 2)]^2$$

Conveting each element to the perfect-square trinomial, and solving the equiation we have that:

$$S(\beta 1, \beta 2) = 4\beta 1^2 + 30 \beta 2 ^2 + 20 \beta 1 \beta 2 - 56 \beta 1 - 154 \beta 2 + 120$$

The minimum is determined by calculating the partial derivatives of $S(\beta 1, \beta 2)$  with respect to $\beta 1$ and $\beta 2$ and setting them to zero

$$\frac{\partial S}{\partial \beta 1} = 8\beta 1 + 20\beta 2 - 56$$


$$\frac{\partial S}{\partial \beta 2} = 20\beta 1 + 60\beta 2 - 154$$

This results in a system of two equations in two unknowns, called the normal equations, which give, when solved


$$\beta1 = 3.5$$

$$\beta2 = 1.4$$


[Return to the main article](/techtalk/machine_learning)
