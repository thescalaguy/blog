---
title: Programming Puzzles 3
tags:
  - epi
date: 2024-11-04 09:06:57
---


In this post, and hopefully in the next few posts, I'd like to devle into the topic of dynamic programming. The aim is to develop an intuition for when it is applicable by solving a few puzzles. I'll be referring to the chapter on dynamic programming in 'Introduction to Algorithms' by CLRS, and elucidating it in my own words.

# Dynamic Programming  

The chapter opens with the definition of dynamic programming: it is a technique for solving problems by combining solutions to subproblems. The subproblems may have subsubproblems that are common between them. A dynamic programming algorithm solves these subsubproblems only once and saves the result, thereby avoiding unnecessary work. The term "programming" refers to a tabular method in which the results of subsubproblems are saved in a table and reused when the same subsubproblem is encountered again.

All of this is abstract so let's look at a concrete example of computing the Fibonacci series.  

{% code lang:python %}
def fibonacci(n: int) -> int:
    return n if n < 2 else fibonacci(n - 1) + fibonacci(n - 2)
{% endcode %}

The call graph for `fibonacci(4)` is given below.

{% asset_img Fibonacci.png %}

As we can see, we're computing `fibonacci(2)` twice. In other words, the subsubproblem of computing `fibonacci(2)` is shared between `fibonacci(4)` and `fibonacci(3)`. A dynamic programming algorithm would solve this subsubproblem only once and save the result in a table and reuse it. Let's see what that looks like.  

{% code lang:python %}
def fibonacci(n: int) -> int:
    T = [0, 1] + ([None] * (n - 2))
    return fibonacci_helper(n=n - 1, T=T)


def fibonacci_helper(n: int, T: list[int | None]) -> int:
    if T[n] is None:
        T[n] = fibonacci_helper(n - 1, T) + fibonacci_helper(n - 2, T)
    return T[n]
{% endcode %}  

In the code above, we create a table `T` which stores the Fibonacci numbers. If an entry exists in the table, we return it immediately. Otherwise, we compute and store it. This recursive approach, with results of subsubproblems stored in a table, is called "top-down with memoziation"; the table is called the "memo". We begin with the original problem and then proceed to solve it by finding solutions to smaller subproblems. The procedure which computes the solution is said to be "memoized" as it remembers the previous computations.  

Another approach is called "bottom-up" in which the solutions to the smaller subproblems are computed first. This depends on the notion that subproblems have "size". In this approach, a solution to the subproblem is found only when the solutions to its smaller subsubproblems have been found. We can apply this approach when computing the Fibonacci series.  

{% code lang:python %}
def fibonacci(n: int) -> int:
    T = [0, 1] + ([None] * (n - 2))

    for i in range(2, n):
        T[i] = T[i - 1] + T[i - 2]

    return T[n - 1]
{% endcode %}  

As we can see, the larger numbers in the Fibonacci series are computed only when the smaller numbers have been computed.   

This was a small example of how dynamic programming algorithms work. They are applied to problems where subproblems share subsubproblems. The solutions to these subsubproblems are stored in a table and reused when they are encountered again. This enables the algorithm to work more efficiently as it avoids the rework of solving the subsubproblems.  

In the next post we'll look at another example of dynamic programming that's presented in the book and implement it in Python to further our understanding of the subject.