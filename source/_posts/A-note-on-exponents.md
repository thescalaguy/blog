---
title: A note on exponents
tags:
  - math
date: 2024-06-17 08:40:07
---


In the chapter on exponents the authors mention that if a base is raised to both a power and a root, we should calculate the root first and then the power. This works perfectly well. However, reversing the order produces correct results, too. In this post we'll see why that works using the properties of exponents.

Let's say we have a base $b$ that is raised to power $p$ and root $r$. We could write this as $b ^ {\frac {p} {r}}$. From the properties of exponents, we could rewrite this as $(b ^ {\frac {1} {r}}) ^ {p}$. Alternatively, it can be written as $b ^ {\frac {1} {r} \times p}$. Since multiplication is commutative, we can switch the order of operations and rewrite it as $(b ^ p) ^ {\frac {1} {r}}$. This means we can calculate the power first and then take the root. 

Let's take a look at a numerical example. Consider $2 ^ {\frac {6} {2}}$. We know that the answer should be equal to $2 ^ 3$. If we were to calculate the root first, we get $(\sqrt {2}) ^ 6 = 8$. If we were to calculate the power first and then take the root, we'd get $(64) ^ \frac {1} {2} = 8$. As we can see, we get the same result.  

Therefore, we can apply the operations in any order.