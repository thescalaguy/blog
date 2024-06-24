---
title: Programming Puzzles 1
tags:
  - epi
date: 2024-06-24 12:28:05
---


I am working my way through a classic book of programming puzzles. As I work through more of these puzzles, I'll share what I discover to solidify my understanding and help others who are doing the same. If you've ever completed puzzles on a site like [leetcode](https://leetcode.com/), you'll notice that the sheer volume of puzzles is overwhelming. However, there are patterns to these puzzles, and becoming familiar with them makes it easier to solve them. In this post we'll take a look at one such pattern - two pointers - and see how it can be used to solve puzzles involving arrays.  

## Two Pointers  

The idea behind two pointers is that there are, as the name suggests, two pointers that traverse the array, with one pointer leading the other. Using these two pointers we update the array and solve the puzzle at hand. As an illustrative example, let us consider the puzzle where we're given an array of even and odd numbers and we'd like to move all the even numbers to the front of the array.  

### Even and Odd

{% code lang:python %}
def even_odd(A: list[int]) -> None:
    """Move even numbers to the front of the array."""
    write_idx = 0
    idx = 0

    while idx < len(A):
        if A[idx] % 2 == 0:
            A[write_idx], A[idx] = A[idx], A[write_idx]
            write_idx = write_idx + 1

        idx = idx + 1
{% endcode %}  

The two pointers here are `idx` and `write_idx`. While `idx` traverses the array and indicates the current element, `write_idx` indicates the position where the next even number should be written. Whenever `idx` points to an even number, it is written at the position indicated by `write_idx`. With this logic, if all the numbers in the array are even, `idx` and `write_idx` point to the same element i.e. the number is swapped with itself and the pointers are moved forward.  

We'll build upon this technique to remove duplicates from the array.  

### Remove Duplicates  

Consider a sorted array containing duplicate numbers. We'd like to keep only one occurrence of each number and overwrite the rest. This can be solved using two pointers as follows.  

{% code lang:python %}
def remove_duplicates(A: list[int]) -> int:
    """Remove all duplicates in the array."""
    write_idx, idx = 1, 1

    while idx < len(A):
        if A[write_idx - 1] != A[idx]:
            A[write_idx] = A[idx]
            write_idx = write_idx + 1

        idx = idx + 1

    return write_idx
{% endcode %}  

In this solution, `idx` and `write_idx` start at index 1 instead of 0. The reason is that we'd like to look at the number to the left of `write_idx`, and starting at index 1 allows us to do that. Notice also how we're writing the `if` condition to check for duplicity in the vicinity of `write_idx`; the number to the left of `write_idx` should be different from the one that `idx` is presently pointing to.  

As a varitation, move the duplicates to the end of the array instead of overwriting them.

{% spoiler Move Duplicates to the End %}

{% code lang:python %}
def remove_duplicates(A: list[int]) -> None:
    """Remove all duplicates in the array and move them to the end"""
    write_idx, idx = 1, 1

    while idx < len(A):
        if A[write_idx - 1] != A[idx]:
            A[write_idx], A[idx] = A[idx], A[write_idx]
            write_idx = write_idx + 1

        idx = idx + 1

    return write_idx
{% endcode %}

{% endspoiler %}

As another variation, remove a given number from the array by moving it to the end.


{% spoiler Remove Key from Array %}

{% code lang:python %}
def remove(A: list[int], k: int) -> None:
    write_idx = 0
    idx = 0

    while idx < len(A):
        if A[idx] != k:
            A[write_idx], A[idx] = A[idx], A[write_idx]
            write_idx = write_idx + 1

        idx = idx + 1
{% endcode %}

{% endspoiler %}

With this same pattern, we can now change the puzzle to state that we want at most two instances of the number in the sorted array. 

### Remove Duplicates Variation  

{% code lang:python %}
def remove_duplicates(A: list[int]) -> int:
    """Keep at most two instances of the number."""
    write_idx = 2

    for idx in range(2, len(A)):
        if A[write_idx - 2] != A[idx]:
            A[write_idx] = A[idx]
            write_idx = write_idx + 1

    return write_idx
{% endcode %}  

Akin to the previous puzzle, we look for duplicates in the vicinity of `write_idx`. While in the previous puzzle the `if` condition checked for one number to the left, in this variation we look at two positions to the left of `write_idx` to keep at most two instances. The remainder of the logic is the same. As a variation, try keeping at most three instances of the number in the sorted array. 

Finally, we'll use the same pattern to solve the [Dutch national flag](https://en.wikipedia.org/wiki/Dutch_national_flag_problem) problem.  

### Dutch National Flag  

In this problem, we sort the array by dividing it into three distinct regions. The first region contains elements less than the pivot, the second region contains elements equal to the pivot, and the third region contains elements greater than the pivot. 

{% code lang:python %}
def dutch_national_flag(A: list[int], pivot_idx: int) -> None:
    """Divide the array into three distinct regions."""
    pivot = A[pivot_idx]
    write_idx = 0
    idx = 0

    # --- Move all elements less than pivot to the front
    while idx < len(A):
        if A[idx] < pivot:
            A[write_idx], A[idx] = A[idx], A[write_idx]
            write_idx = write_idx + 1
        idx = idx + 1

    idx = write_idx

    # -- Move all elements equal to the pivot to the middle
    while idx < len(A):
        if A[idx] == pivot:
            A[write_idx], A[idx] = A[idx], A[write_idx]
            write_idx = write_idx + 1
        idx = idx + 1

    # -- All elements greater than pivot have been moved to the end.
{% endcode %}  

This problem combines everything we've seen so far about two pointers and divides the array into three distinct regions. As we compute the first two regions, the third region is computed as a side-effect.  

We can now solve a variation of the Dutch national flag partitioning problem by accepting a list of pivot elements. In this variation all the numbers within the list of pivots appear together i.e. all the elements equal to the first pivot element appear first, equal to second pivot element appear second, and so on.  

{% code lang:python %}
def dutch_national_flag(A: list[int], pivots: list[int]) -> None:
    """This is a variation in which all elements with same key appear together."""
    write_idx = 0

    for pivot in pivots:
        idx = write_idx

        while idx < len(A):
            if A[idx] == pivot:
                A[write_idx], A[idx] = A[idx], A[write_idx]
                write_idx = write_idx + 1
            idx = idx + 1
{% endcode %}  

That's it, that's how we can solve puzzles involving two pointers and arrays.