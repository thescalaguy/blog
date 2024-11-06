---
title: Programming Puzzles 4
tags:
  - epi
date: 2024-11-06 11:46:05
---


In the previous post we looked at computing the Fibonacci series both with and without dynamic programming. In this post we'll look at another example where dynamic programming is applicable. The example is borrowed from 'Introduction to Algorithms' by CLRS and implemented in Python. By the end of this post we'll try to develop an intuition for when dynamic programming applies.

# Rod Cutting  

The problem we are presented with is the following: given a steel rod, we'd like to find the optimal way to cut it into smaller rods. More formally, we're presented with a rod of size n inches and a table of prices p<sub>i</sub>. We'd like to determine the maximum revenue r<sub>n</sub> that we can obtain by cutting the rod and selling it. If the price p<sub>n</sub> of the rod of length n is large enough, we may sell the rod without making any cuts. 

The table of prices that we'll work with is given below.  


| length i   | 1  | 2  | 3  | 4  | 5  | 6  | 7  | 8  | 9  | 10  |
|---|---|---|---|---|---|---|---|---|---|---|
| **price p<sub>i</sub>**   | 1  | 5  | 8  | 9  | 10  | 17  | 17  | 20  | 24  | 30  |  


Consider a rod of length 4 inches. The maxium revenue we can obtain is 10 by cutting the rod into two parts of length 2 inches each.

{% asset_img RodCut.png %}

Given a rod of n inches, we may sell it uncut or we may sell it by cutting it into smaller pieces. Since we do not know the size of the cuts to make, we will have to consider all possible sizes. Once we make a cut of size $i$ from the left end of the rod, we can view the remaining length of the rod of size $n - i$ as an independent instance of the rod cutting problem. In other words, we are solving a smaller instance of the same problem. The equation below shows how we can mathematically formulate the problem.  

<center>
{% mathjax %}
r_n = \underset {1 \le i \le n} {\text{max}} (p_i + r_{n - i})
{% endmathjax %}
</center>

It states that the revenue r<sub>n</sub> is the maximum revenue obtained by considering all cuts of size $i = 1, 2, ... n$ plus the revenue obtained by cutting the remaining rod of size $n - i$. We can write a recursive function to obtain this value as follows:  

{% code lang:python %}
def rod_cut(p: list[int], n: int) -> int:
    if n == 0:
        return 0

    q = float("-inf")

    for i in range(1, n + 1):
        q = max(q, p[i] + rod_cut(p, n - i))

    return q
{% endcode %}  

We can verify the results by calling the function for a rod of size 4 and passing the table of prices.  

{% code lang:python %}
p = [0, 1, 5, 8, 9, 10, 17, 17, 20, 24, 30]
assert 10 == rod_cut(p=p, n=4)
{% endcode %}  

The recursive version, however, does redundant work. Consider the rod of size n = 4. We will have to consider cuts of size $i = 1, 2, 3, 4$. When considering the remaining rod of size 3, we'd consider cuts of size $i = 1, 2, 3$. In both of these cases we recompute, for example, the revenue obtained when the remainder of the rod is of size 2.  

We can use dynamic programming to solve this problem by modifying the `rod_cut` function as follows.

{% code lang:python %}
def rod_cut(p: list[int], t: list[int | None], n: int) -> int:
    if n == 0:
        return 0

    if t[n] is not None:
        return t[n]

    q = float("-inf")

    for i in range(1, n + 1):
        q = max(q, p[i] + rod_cut(p, t, n - i))

    t[n] = q

    return q
{% endcode %}

Notice how we've introduced a table `t` which stores the maximum revenue obtained by cutting a rod of size n. This allows us to reuse previous computations. We can run this function and verify the result.

{% code lang:python %}
t = [None] * 11
p = [0, 1, 5, 8, 9, 10, 17, 17, 20, 24, 30]
assert 10 == rod_cut(p, t, 4)
{% endcode %}   

The question that we're left with is the following: how did we decide that the problem could be solved with dynamic programming? There are two key factors that help us in deciding if dynamic programming can be used. The first is **overlapping subproblems** and the second is **optimal substructure**.  

For a dynamic programming algorithm to work, the number of subproblems must be small. This means that the recursive algorithm which solves the problem encounters the same subproblems over and over again. When this happens, we say that the problem we're trying to solve has overlapping subproblems. In the rod cutting problem, when we try to cut a rod of size 4, we consider cuts of size $i = 1, 2, 3, 4$. When we, then, consider the remaining rod of size 3, we consider cuts of size $i = 1, 2, 3$. The smaller problem of optimally cutting the rod of size 2 is encountered again. In other words, it is an overlapping subproblem. Dynamic programming algorithms solve an overlapping subproblem once and store its result in a table so that it can be reused again.  

The second factor is optimal substructure. When a problem exhibits optimal substructure, it means that the solution to the problem contains within it the optimal solutions to the subproblems; we build an optimal solution to the problem from optimal solutions to the subproblems. The rod-cutting problem exhibits optimal substructure because the optimal solution to cutting a rod of length $n$ involves finding the optimal solution to cutting the remaining rod, if a cut has been made.  

Moving on. So far our algorithm has returned the optimal value of the solution. In other words, it returned the maximum revenue that can be obtained by optimaly cutting the rod. We often need to store the choice that led to the optimal solution. In the context of the rod cutting problem, this would be the lengths of the cuts made. We can do this by keeping additional information in a separate table. The following code listing is a modification of the above function with an additional table `s` to store the value of the optimal cut.  

{% code lang:python %}
def rod_cut(p: list[int], s: list[int | None], t: list[int | None], n: int) -> int:
    if n == 0:
        return 0

    if t[n] is not None:
        return t[n]

    q = float("-inf")
    j = None

    for i in range(1, n + 1):
        r = p[i] + rod_cut(p, s, t, n - i)

        if r > q:
            q = r
            j = i

    s[n] = j
    t[n] = q

    return q
{% endcode %}  

In this version of the code, we store the size of the cut being made for a rod of length `n` in the table `s`. Once we have this information, we can reconstruct the optimal solution. The function that follows shows how to do that.  

{% code lang:python %}
def optimal_cuts(s: list[int | None], n: int) -> list[int]:
    cuts = []

    while n:
        cut = s[n]
        n = n - s[n]
        cuts.append(cut)

    return cuts
{% endcode %}  

Finally, we call the function to see the optimal solution. Since we know that the optimal solution for a rod of size 4 is to cut it into two equal halves, we'll use $n = 4$. 

{% code lang:python %}
n = 4
t = [None] * 11
s = [None] * 11
p = [0, 1, 5, 8, 9, 10, 17, 17, 20, 24, 30]
q = rod_cut(p, s, t, n)

assert [2, 2] == optimal_cuts(s, n)
{% endcode %}  

That's it. That's how we can use dynamic programming to optimally cut a rod of size $n$ inches.