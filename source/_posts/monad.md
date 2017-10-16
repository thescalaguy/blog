---
title: Monad
date: 2017-10-13 06:20:09
tags:
  - category theory
  - scala
  - functional programming
---
So far we've looked at monoids and functors. The next algebraic data structure we'll cover is a monad. If you've wondered what a monad is but never really understood it, this is the post for you. I am sure that you've used it without realizing it. So let's get to it.

## Definition

A monad has more structure than a functor. This means that you can call `map` on it and that it obeys all the functor laws. In addition, a monad has a `flatMap` function which you can use to chain monads together. In essence, monads represent units of computation that you can chain together and the result of this chaining is also a monad.

Let's look at a few examples.

## Example

{% codeblock lang:scala %}
val first = List(1, 2)
val next = List(8, 9)

for {
  i <- first
  j <- next
}
yield(i * j)
{% endcodeblock %}

The above code<sup>[[1]](http://debasishg.blogspot.in/2008/03/monads-another-way-to-abstract.html)</sup> uses a `for` comprehension to muliply elements of the list together. Under the hood, this gets translated to:

{% codeblock lang:scala %}
first flatMap {
  f => next map {
    n => f * n
  }
}
{% endcodeblock %}

The compiler is making use of the `List` monad to chain operations together. Let's break this down. 

{% codeblock lang:scala %}
next map {
  n => f * n
}
{% endcodeblock %}

This part of the code will return a `List` since that is what calling `map` on a `List` does. Since we have two elements in `first` list, the result of mapping will generate two lists of two elements each. This isn't what we want. We want a single list that combines the results together. 

{% codeblock lang:scala %}
first flatMap {
  ...
}
{% endcodeblock %}

The flattening of results is what `flatMap` does - it takes the two lists and squishes them into one.

## Monad Laws

For something to be a monad, it has to obey the monadic laws. There's three monad laws:  

1. Left identity
2. Right identity
3. Associativity

### Left Identity

This law means that if we take a value, put it into a monad, and then `flatMap` it with a function `f`, that's the same as simply applying the function `f` to the original value. Let's see this in code:

{% codeblock lang:scala %}
scala> def f(x: Int): List[Int] = { List(x * 2) }
f: (x: Int)List[Int]

// left identity
List(2).flatMap(f) == f(2)
res5: Boolean = true
{% endcodeblock %}

### Right Identity

This law means that if we take a monad, `flatMap` it, and within that `flatMap` we try to create a monad out of it, then that's the same as original monad. Let's see this in code:

{% codeblock lang:scala %}
// right identity
scala> List(1, 2, 3).flatMap({ x => List(x) }) == List(1, 2, 3)
res6: Boolean = true
{% endcodeblock %}

Let's walkthrough this. The function to `flatMap` gets the elements of the original list, `List(1, 2, 3)`, one-by-one. The result is `List(List(1), List(2), List(3))`. This is then flattened to create `List(1, 2, 3)`, which is the original list.

### Associativity

This law states that if we apply a chain of functions to our monad, that's the same as the composition of all the functions. Let's see this in code:

{% codeblock lang:scala %}
scala> def f(x: Int): List[Int] = { List(x + 1) }
f: (x: Int)List[Int]

scala> def g(x: Int): List[Int] = { List(x + 1) }
g: (x: Int)List[Int]

scala> List(1, 2, 3).flatMap(f).flatMap(g) == List(1, 2, 3).flatMap(x => f(x).flatMap(g))
res8: Boolean = true
{% endcodeblock %}

## Conclusion

This brings us to the end of the post on monads and their laws. `List` isn't the only monad in your arsenal. `Option`s and `Future`s are monads, too. I suggest going ahead and constructing examples for monadic laws for them. 