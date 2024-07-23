---
title: A question on algebraic manipulations
tags:
  - math
date: 2024-07-23 12:19:29
---


In the exercises that follow the chapter on algebraic manipulations, there is a question that pertains to expressing an integer $q$ as the sum of two other integers squared. We are then asked to find expressions for the multiples of $q$, namely $2q$ and $5q$. In this post, we'll take a look at the solution provided by the authors for the first half of the question, $2q$, and then come up with a method of our own to solve the second half, $5q$.  

## Question  

If $q$ is an integer that can be expressed as the sum of two integer squares, show that both $2q$ and $5q$ can also be expressed as the sum of two integer squares.  

## Solution  

From the question, $q = a ^ 2 + b ^ 2$ since it is the sum of two integer squares. This means $2q = 2 (a ^ 2 + b ^ 2) = 2 a ^ 2 + 2 b ^ 2$. We need to find two integers such that when their squares are summed, we end with $2q$. From the solution, these are the numbers (a + b) and (a - b) because when they are squared and summed, we get $2a ^ 2 + 2b ^ 2$. This is the result of $(a + b) ^ 2 + (a - b) ^ 2$. Go ahead and expand them to verify the result.  

What do we deduce from this? We find that both the expressions contributed an $a ^ 2$ and a $b ^ 2$. These were added together to get the final result. How do we use this to get $5q = 5 a ^ 2 + 5 b ^ 2$? Notice that we're squaring the integers. This means that, for example, one of them would have to contribute an $a ^ 2$ and the other would have to contribute a $4 a ^ 2$; similar logic applies for $5 b ^ 2$.  

This leaves us with two pairs of numbers -- $(a + 2b), (2a - b)$ and $(2a + b), (a - 2b)$. Let's square and sum both of these numbers one-by-one.  

$(a + 2b) ^ 2 + (2a - b) ^ 2 = (a ^ 2 + 4ab + 4b ^ 2) + (4a ^ 2 - 4ab + b ^ 2) = 5a ^ 2 + 5b ^ 2$  

$(2a + b) ^ 2 + (a - 2b) ^ 2 = (4a ^ 2 + 4ab + b ^ 2) + (a ^ 2 - 4ab + 4b ^ 2) = 5a ^ 2 + 5b ^ 2$  


What integers would we need for $10q = 10a ^ 2 + 10b ^ 2$?