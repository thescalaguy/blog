---
title: Factoring Quadratics
tags:
  - math
date: 2024-06-10 12:05:58
---


While reading through the chapter on solving quadratics of a math textbook, I came across a paragraph where the authors mention that factoring quadratics takes a bit of ingenuity, experience, and dumb luck. I spent some time creating an alternative method from the one mentioned in the book which makes factoring quadratics a matter of following a simple set of steps, and removes the element of luck from it. In this post I will review some of the concepts mentioned in the book, and solve through one of the more difficult problems to illustrate my method.

## Concepts  

Let's begin by looking at the quadratic $x ^ 2 - 3x + 2$. This can be factored as $(x + u)(x + v)$. Multiplying the terms gives us $x ^ 2 + (u + v)x + uv$. Comparing this to the coefficients of the the original quadratic gives us $u + v = -3$, and $uv = 2$. For a simple quadratic, we can guess that $u = -1$ and $v = -2$. For quadratics where it is not so obvious, we need hints to guide us along the way.  

We can get insights into the signs of $u$ and $v$ by looking at the product and the sum of coefficients of the quadratic. If the product is positive, then they have the same signs. If the product is negative, they have different signs. This makes intuitive sense. In case the product is positive, the sum tells us whether they are both positive or both negative.  

We will use this again when we look at the alternative method to factor a quadratic. First, however, we will look at a different type of quadratic where the coefficient of $x ^ 2$ is not 1.  

Consider the quadratic $4 x ^ 2 + 8 x + 3$. It can be factored as $(sx + u)(tx + v)$. Multiplying the terms gives us $stx^2 + (sv + ut)x + uv$. As in the previously mentioned quadratic, we can get the product and the sum terms by comparing the quadratic with the general form we just derived. Here $st = 4$, $sv + ut = 8$ and $uv = 3$. A small nuance to keep in mind is that if the coefficient of $x ^ 2$ were negative, we'd factor out a $-1$ to make it positive.

Now we move on to the problem and the method. You'll find that although the method is tedious, it will remove the element of luck from the process.

## Method

Consider the quadratic $49 x ^ 2 - 316x + 132$. Here, $st = 49$, $sv + ut = - 316$, and $uv = 132$. From the guiding hints mentioned in the previous section, we notice that the signs of $u$ and $v$ are the same; they are either both positive or both negative. We begin the method be defining a function $\text{F}(n)$ which returns a set of pairs of all the factors of $n$. Therefore, we can write $\text{F}(st)$ and $\text{F}(uv)$ as follows.

<p align="center"> $\text{F}(st) = \text{F}(49) = \{ (1,49), (7,7) \}$ </p>  

<p align="center"> $\text{F}(uv) = \text{F}(132) = \{ (1,132), (2, 66), (3, 44), (4, 33), (6, 22) \}$ </p>  

We will now get to the tedious part. We will create combinations of $s, t, u, v$. We pick the first pair of factors of $st$ and match it with the first pair of factors of $uv$. For the sake of brevity, we will only some of the examples to illustrate the process. 

| $s$  | $t$  | $u$  | $v$  | $sv + ut$  |
|---|---|---|---|---|
| 7  | 7  | 2  | 66  | 476  |
| 7  | 7  | -2  | -66  | -476  |
| 7   | 7 | 66 | 2  |  476 |
| 7  |  7 | -66  | -2  | -476  |  

Notice how we swap the values of $u$ and $v$ in the columns above. This is because we're trying to find the product $sv + ut$ and its value will change depending on what the value of $u$ and $v$ are. Similarly, we'll have to swap the values of $s$ and $t$; this is not apparent in the table above because both the values are $7$. We will have to continue on with the table since we we are yet to find a combination of numbers which equals $-316$. The table is quite large so we'll skip the rest of the entries for the sake of brevity and look at the one which gives us the value we're looking for.  

| $s$  | $t$  | $u$  | $v$  | $sv + ut$  |
|---|---|---|---|---|
| 49  | 1  | -22  | -6  | -316  |  

We can now write our factors as $(49 x - 22)(x - 6)$. This gives us $x = 6$ or $x = \frac {22} {49}$.   

That's it. That's how we can factor quadratics by creating combinations of the factors of their coefficients.