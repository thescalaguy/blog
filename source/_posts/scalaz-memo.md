---
title: Scalaz Memo
date: 2017-08-05 23:42:31
tags:
  - scalaz
  - scala
  - functional programming
---
In this post we'll look at [Memo](https://github.com/scalaz/scalaz/blob/fab6f1f97664590652bf37d64750e229c0524000/core/src/main/scala/scalaz/Memo.scala) which is a Scalaz goodie to add memoization to your program. We'll recap what memoization is, write a recursive function to calculate Fibonacci numbers, and then add memoization to it using `Memo`.  

## What is Memoization?

In computing, memoization or memoisation is an optimization technique used primarily to speed up computer programs by storing the results of expensive function calls and returning the cached result when the same inputs occur again.<sup>[[1]](https://en.wikipedia.org/wiki/Memoization)</sup> 

## Fibonacci (without Memo)

{% codeblock lang:scala %}
@ def fibo(n: Int): Int = {
    n match {
      case 0 | 1 => n
      case _: Int => fibo(n - 1) + fibo(n - 2)
    }
  }
defined function fibo
{% endcodeblock %}

So here's the non-memoized recursive `fibo` which calculates the n<sup>th</sup> Fibonacci number. The issue here is that it'll recalculate the Fibonacci numbers a lot of times and therefore cannot be used for large values of n. 

## Fibonacci (with Memo)

{% codeblock lang:scala %}
@ import scalaz._
import scalaz._
@ import Scalaz._
import Scalaz._
@ val fibo: Int => Int = Memo.immutableHashMapMemo {
    case 0 => 0
    case 1 => 1
    case n: Int => fibo(n - 1) + fibo(n - 2)
  }
fibo: Int => Int = scalaz.Memo$$$Lambda$2233/1055106802@48649202
{% endcodeblock %} 

Here's the memoized version using Scalaz `Memo`. We are using an immutable, hash map-backed memo. The `immutableHashMapMemo` method takes a partial function defining how we construct the memo. In our case, if the value of n is not 0 or 1, we try looking up the value in the memo again. We recurse until we reach 0 or 1. Once that happens and our recursion returns, the resultant value is cached in an `immutable.HashMap`.

## Conclusion 

Memoization is a great way to optimize your programs. In this post we used an immutable hash map memo. There are other types of memos applying different strategies to cache their results. The `Memo` companion object is the place to look for an appropriate memo that suits your needs.