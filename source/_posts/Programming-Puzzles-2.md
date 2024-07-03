---
title: Programming Puzzles 2
tags:
  - epi
date: 2024-07-03 17:26:27
---


As I continue working my way through the book on programming puzzles, I came across those involving permutations. In this post I'll collect puzzles with the same theme, both from the book and from the internet.  

## All permutations  

The first puzzle is to compute all the permutations of a given array. By extension, it can be used to compute all the permutations of a string, too, if we view it as an array of characters. To do this we'll implement [Heap's algorithm](https://en.wikipedia.org/wiki/Heap%27s_algorithm). The following is its recursive version.  

{% code lang:python %}
def heap(permutations: list[list], A: list, n: int):
    if n == 1:
        permutations.append(list(A))
    else:
        heap(permutations, A, n - 1)

        for i in range(n - 1):
            if n % 2 == 0:
                A[i], A[n - 1] = A[n - 1], A[i]
            else:
                A[0], A[n - 1] = A[n - 1], A[0]

            heap(permutations, A, n - 1)
{% endcode %}  

The array `permutations` is the accumulator which will store all the permutations of the array. The initial arguments to the function would be an empty acuumulator, the list to permute, and the length of the list.

## Next permutation

The next puzzle we'll look at is computing the next permutation of the array in lexicographical order. The following implementation has been taken from the book.  

{% code lang:python %}
def next_permutation(perm: list[int]) -> list[int]:
    inversion_point = len(perm) - 2

    while (inversion_point >= 0) and (perm[inversion_point] >= perm[inversion_point + 1]):
        inversion_point = inversion_point - 1

    if inversion_point == -1:
        return []

    for i in reversed(range(inversion_point + 1, len(perm))):
        if perm[i] > perm[inversion_point]:
            perm[inversion_point], perm[i] = perm[i], perm[inversion_point]
            break

    perm[inversion_point + 1:] = reversed(perm[inversion_point + 1:])

    return perm
{% endcode %}  

## Previous permutation

A variation of the puzzle is to compute the previous permutation of the array in lexicographical order. The idea is to "reverse" the logic for computing the next permutation. If we look closely, we'll find that all we're changing are the comparison operators.

{% code lang:python %}
def previous_permutation(perm: list[int]) -> list[int]:
    inversion_point = len(perm) - 2

    while (inversion_point >= 0) and (perm[inversion_point] <= perm[inversion_point + 1]):
        inversion_point = inversion_point - 1

    if inversion_point == -1:
        return []

    for i in reversed(range(inversion_point + 1, len(perm))):
        if perm[i] < perm[inversion_point]:
            perm[inversion_point], perm[i] = perm[i], perm[inversion_point]
            break

    perm[inversion_point + 1:] = reversed(perm[inversion_point + 1:])

    return perm
{% endcode %}  

## kth smallest permutation

The final puzzle we'll look at is the one where we need to compute the k'th smallest permutation. The solution to this uses the `previous_permutation` function that we saw above. The idea is to call this function k times on the lexicographically-largest array. Sorting the array in decreasing order results is the largest. This becomes the input to the `previous_permutation` function.

{% code lang:python %}
def kth_smallest_permutation(perm: list[int], k: int) -> list[int]:
    # -- Arrange the numbers in decreasing order
    # -- thereby creating the lexicographically largest permutation
    perm = sorted(perm, reverse=True)

    for _ in range(k):
        perm = previous_permutation(perm)

    return perm
{% endcode %}  

That's it. These are puzzles involving permutations.