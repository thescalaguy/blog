---
title: Monoid
date: 2017-08-10 11:16:14
tags:
  - category theory
  - scala
  - functional programming
---

The first algebraic structure we'll look at is a monoid. We've covered monoid previously in [Scalaz Under the Hoods](http://fasihkhatib.com/2017/06/24/scalaz-under-the-hoods/). In this post we'll look at it again from a more abstract standpoint.

## Formal Definition

A monoid $(M, \bullet, e)$ is an underlying $M$ equipped with
1. a binary operation $\bullet$ from pairs of elements of $M$ into $M$ such that $(x \bullet y) \bullet z = x \bullet (y \bullet z)$ for all $x, y, z \in M$
2. an element $e$ such that $e \bullet x = x = x \bullet e$  

We've already translated this definition to code. Just to recap, here's what we wrote previously:  

{% codeblock lang:scala %}
trait Monoid[A] {
  def mempty: A
  def mappend(a: A, b: A): A
}
{% endcodeblock %}  

`mappend` is the binary operation $\bullet$, and `mempty` is the element $e$.  

More concretely, we wrote this:

{% codeblock lang:scala %}
object IntMonoid extends Monoid[Int] {
  def mempty: Int = 0
  def mappend(a: Int, b: Int) = a + b
}
{% endcodeblock %}

So, $\bullet$ translates to the addition operation $+$, and $e$ translates to $0$. That way, $0 + x = x = x + 0$ where $x$ is any integer. That was fairly easy to understand.

## Monoid Homomorphism

A monoid homomorphism from $(M, \bullet, e)$ to $(M^\prime, \bullet^\prime, e^\prime)$ is a function $f: M \rightarrow M^\prime$ such that 

1. $f(e) = e^\prime$ and 
2. $f(x \bullet y) = f(x) \bullet^\prime f(y)$.   

The composition of two monoid homomorphisms is the same as their composition as functions on sets.  

I know this is abstract so let's have a look at a concrete example. Let's write some code. We'll be reusing the monoids that we previously wrote.

{% codeblock lang:scala %}
@ trait Monoid[A] {
    def mempty: A
    def mappend(a: A, b: A): A
  }
defined trait Monoid

@ object Monoid {
    implicit object StringMonoid extends Monoid[String] {
        def mempty: String = ""
        def mappend(a: String, b: String): String = a + b
    }
    implicit object IntMonoid extends Monoid[Int] {
        def mempty: Int = 0
        def mappend(a: Int, b: Int) = a + b
    }
  }
defined object Monoid
{% endcodeblock %}

Next, we'll write a homomorphism $f: M \rightarrow M^\prime$

{% codeblock lang:scala %}
@ def f(s: String): Int = s length
defined function f
{% endcodeblock %}

Let's see this in action. We'll begin by testing the first rule.

{% codeblock lang:scala %}
// bring the monoids into scope
@ import Monoid._
import Monoid._

// rule 1
@ f(StringMonoid.mempty) == IntMonoid.mempty
res6: Boolean = true
{% endcodeblock %}

So we see that the first rule is satisfied. Applying $f$ on the zero element of `StringMonoid` gives us the zero element of `IntMonoid`. Onto the second rule.

{% codeblock lang:scala %}
@ val x = "apple"
x: String = "apple"
@ val y = "banana"
y: String = "banana"

// rule 2
@ f( StringMonoid.mappend(x, y) ) == IntMonoid.mappend( f(x), f(y) )
res9: Boolean = true
{% endcodeblock %}

And we see that the second rule is also satisfied. Therefore, $f$ is a homomorphism such that $f: StringMonoid \rightarrow IntMonoid$. To recap, a monoid homomorphism is a map between monoids that preserves the monoid operation and maps the identity element of the first monoid to that of the second monoid<sup>[[1]](https://en.wikipedia.org/wiki/Homomorphism#Definition)</sup>. The monoid operation is still $+$ and the empty string is mapped to $0$, which is the zero/identity element of `IntMonoid`. 