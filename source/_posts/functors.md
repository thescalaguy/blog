---
title: Functor
date: 2017-09-05 08:46:09
tags:
  - category theory
  - scala
  - functional programming
---

The next algebraic structure we'll look at is a functor. In the [introduction](http://fasihkhatib.com/2017/08/08/category-theory-intro/), we saw that a category consists of objects and arrows. As an example, we morphed a set of strings to another set which contained the reverse of those strings. In other words, we morphed an object to another object. What if we could morph an entire category to another category while preserving the structure? Well, that's what a functor does.

## Formal Definition

Let $C$ and $D$ be categories. A **functor** $F: C \rightarrow D$ is a map taking each $C$-object $A$ to a $D$-object $F(A)$ and each $C$-arrow $f: A \rightarrow B$ to a $D$ arrow $F(f): F(A) \rightarrow F(B)$, such that all $C$-objects $A$ and composable $C$-arrows $f$ and $g$ 

<ol>
<li> $F(id_A) = id_{F(A)}$ </li>
<li> $F(g \circ f) = F(g) \circ F(f)$
</ol>

## Example

Say we have a set $S$. From this set we create another set $List(S)$ which contains finite lists of elements drawn from $S$. The functor we want maps from set to set. Since we know that a category contains objects and arrows, $List$ becomes the object part. The arrow part takes a function $f:S \rightarrow S^\prime$ to a function $List(f): List(S) \rightarrow List(S^\prime)$ that given a list $L = [s_1, s_2, s_3, ... s_n]$ maps $f$ over elements of $L$

$$
List(f)(L) = maplist(f)(L) = [f(s_1), f(s_2), f(s_3), ... f(s_n)]
$$ 

How does this translate to code? This actually translates fairly easily to code. Containers like lists, trees, etc. that you can call `map` on are functors.   

Let's write some code. We'll begin by creating a set $S$.

{% codeblock lang:scala %}
@ val S = Set(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
S: Set[Int] = Set(0, 5, 1, 6, 9, 2, 7, 3, 8, 4)
{% endcodeblock %}

Next, we'll create $f: S \rightarrow S^\prime$.

{% codeblock lang:scala %}
@ def f(x: Int) = { x * 2 }
defined function f
{% endcodeblock %}

Next, let's create $L$.

{% codeblock lang:scala %}
@ val L = List(1, 2, 3)
L: List[Int] = List(1, 2, 3)
{% endcodeblock %}

Next, we'll create the function `maplist`.

{% codeblock lang:scala %}
@ def maplist(f: Int => Int)(L: List[Int]) = L map f
defined function maplist
{% endcodeblock %}

Finally, let's see this in action:

{% codeblock lang:scala %}
@ maplist(f)(L)
res4: List[Int] = List(2, 4, 6)
{% endcodeblock %}

As we can see, `maplist` applied the function `f` on all elements of `L`. We did this by using the `map` method of a `List` instance. 

## Functor Laws

All functors are expected to obey the two laws that we saw in the formal definition. Let's see how they translate to code.

### First Law

$$F(id_A) = id_{F(A)}$$

The first law states that if we map the identity function over a functor, we'll get back a functor which is the same as the original functor.

{% codeblock lang:scala %}
@ List(1, 2, 3) map identity
res5: List[Int] = List(1, 2, 3)
{% endcodeblock %}

As we can see, applying `identity` to the list gives back the same list.

### Second Law

$$F(g \circ f) = F(g) \circ F(f)$$

The second law states that if we map a functor using a composition of two functions, $F(g \circ f)$, it's the same as first mapping the functor using the first function and then mapping the resulting functor using the second function, $F(g) \circ F(f)$.

We'll begin by creating two functions `f` and `g`.

{% codeblock lang:scala %}
@ def f(x: Int): Int = x + 1
defined function f
@ def g(x: Int): Int = x + 1
defined function g
{% endcodeblock %}

Now let's put the theory into practice.

{% codeblock lang:scala %}
// composition of two functions
@ List(1, 2, 3) map { x => g(f(x)) }
res8: List[Int] = List(3, 4, 5)

// applying map twice
@ List(1, 2, 3) map f map g
res9: List[Int] = List(3, 4, 5)
{% endcodeblock %}

As we see, the two lists are the same.

## More Functor Examples

### Example 1

Let's consider a category where objects are integers. Arrows between objects indicates a "divided by" relationship. For example,

$$
10 \rightarrow 5
$$

This indicates that 10 can be divided by 5. To reiterate, objects are numbers and arrows represent a "divided by" relationship. 

Now let's create a functor from the category to itself. This functor will multiply each object by 13. So, $F(10) = 130$. Is this a valid functor? We have $a \rightarrow b$ but is it true that $F(a) \rightarrow F(b)$?

The answer is yes. Our category has arrows that indicate a "divided by" relationship. So, $\frac{a}{b}$ will be an integer. Similarly, $\frac{13a}{13b}$ will also be an integer and maintain a "divided by" relationship. This shows that arrows do not always have to be functions. They can also indicate a relationship between their domain and codomain.


## Conclusion

In this post we saw functors which map objects from one category to another. Containers like trees, lists, etc. are functors. All functors are required to obey the two functor laws.
