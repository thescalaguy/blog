---
title: Scalaz Order
date: 2017-06-29 14:30:08
tags:
  - scalaz
  - scala
  - functional programming
---

In this post we'll look at how to implement ordering using Scalaz [`Order`](https://github.com/scalaz/scalaz/blob/fabab8f699d56279d6f2cc28d02cc2b768e314d7/core/src/main/scala/scalaz/Order.scala) trait. However, before we do that, let's step back a little and look at how we implement ordering using [`scala.math.Ordering`](http://www.scala-lang.org/api/current/scala/math/Ordering.html)  

## Sorting a Collection - Scala Way 

Say we have a `List` and we'd like to sort it.<sup>[[1]](http://alvinalexander.com/scala/how-to-sort-scala-collections-sortwith-sorted-ordered-ordering)</sup> We can do so by calling the `sorted` method on it and it'll work out-of-the-box for lists of `Int`, `Float`, etc. because there is an implicit in `scala.math.Ordering`.

{% codeblock lang:scala %}
@ List(1, 2, 3, 5, 4, 10, -1, 0).sorted
res0: List[Int] = List(-1, 0, 1, 2, 3, 4, 5, 10)
{% endcodeblock %}  

Going over the documentation for `scala.math.Ordering`<sup>[[2]](http://www.scala-lang.org/api/current/scala/math/Ordering.html)</sup>:  

*Ordering is a trait whose instances each represent a strategy for sorting instances of a type.
Ordering's companion object defines many implicit objects to deal with subtypes of AnyVal (e.g. Int, Double), String, and others.*  

All the implicit objects in the companion object implement the `scala.math.Ordering` trait which extends the `java.util.Comparator` interface. Thus, they all have a `compare` method. What if we have types for which there is no sorting strategy in the companion object of `scala.math.Ordering`? We define our own like so:

{% codeblock lang:scala %}
@ case class Salary(amt: Float)
defined class Salary
@ implicit object SalaryOrdered extends Ordering[Salary] {
    // we are using compare on type Float
    def compare(a: Salary, b: Salary): Int = a.amt compare b.amt
  }
defined object SalaryOrdered
@ List(Salary(999.0f), Salary(555.0f)).sorted
res4: List[Salary] = List(Salary(555.0F), Salary(999.0F))
{% endcodeblock %}  

But we still don't have operators like `<`, `<=`, etc. on type `Salary`.  

{% codeblock lang:scala %}
@ Salary(999.0f) > Salary(555.0f)
cmd3.sc:1: value > is not a member of ammonite.$sess.cmd0.Salary
val res3 = Salary(999.0f) > Salary(555.0f)
                          ^
Compilation Failed
{% endcodeblock %}  

To do that, the trait `scala.math.Ordered` would have to be mixed in.  

{% codeblock lang:scala %}
@ case class Salary(amt: Float) extends Ordered[Salary] {
    override def compare(that: Salary): Int = this.amt compare that.amt
  }
defined class Salary
@ Salary(1.0f) < Salary(2.0f)
res1: Boolean = true
@ List(Salary(2.0f), Salary(2.0f), Salary(1.0f)).sorted
res2: List[Salary] = List(Salary(1.0F), Salary(2.0F), Salary(2.0F))
{% endcodeblock %}  

And we know that if we're working with a library, we do not have the liberty to mix a trait for our convenience. Also, Scala's comparison operators are not type safe.  

{% codeblock lang:scala %}
@ 1 > 2.0
res4: Boolean = false
{% endcodeblock %}  

## Sorting a Collection - Scalaz Way  

As stated before, Scalaz provides `Order` trait. Staying with our `Salary` example, let's put Scalaz `Order` to use.  

{% codeblock lang:scala %}
@ import scalaz._
import scalaz._
@ import Scalaz._
import Scalaz._
@ case class Salary(amt: Float)
defined class Salary
@ implicit object SalaryOrder extends Order[Salary] {
    override def order(a: Salary, b: Salary): Ordering = {
      a.amt compare b.amt match {
        case -1 => Ordering.LT
        case 0 => Ordering.EQ
        case 1 => Ordering.GT
      }
    }
  }
defined object SalaryOrder
{% endcodeblock %}

We've defined a way to order `Salary` objects using Scalaz `Order` trait via a typeclass. The implicit object needs to provide an implementation for `order` method which returns an [`Ordering`](https://github.com/scalaz/scalaz/blob/fabab8f699d56279d6f2cc28d02cc2b768e314d7/core/src/main/scala/scalaz/Ordering.scala) value. Going over the code for Scalaz `Ordering`: 

*This Ordering is analogous to the Ints returned by scala.math.Ordering.*  

Now, we have comparison operators at our disposal. Here's how we can compare `Salary` objects using the `?|?` operator:

{% codeblock lang:scala %}
@ Salary(1.0f) ?|? Salary(2.0f)
res4: Ordering = LT
{% endcodeblock %}  

`?|?` can also be used with `Int`, `Float`, etc. and is type-safe.  

{% codeblock lang:scala %}
@ 1.0 ?|? 2.0
res5: Ordering = LT
@ 1.0 ?!? 2
cmd6.sc:1: value ?!? is not a member of Double
val res6 = 1.0 ?!? 2
               ^
Compilation Failed
{% endcodeblock %}  

Also, analogous to `<`, `<=`, we now have type-safe `lt`, `lte`, etc.  

{% codeblock lang:scala %}
@ 1.0 gte 2.0
res6: Boolean = false
@ 1 gt 2.0
cmd8.sc:1: type mismatch;
 found   : Double(2.0)
 required: Int
val res8 = 1 gt 2.0
                ^
Compilation Failed
{% endcodeblock %}

That's not where the power ends. You can also compare `Option`s seamlessly. Do note that it is `some` with a lowercase "s".  

{% codeblock lang:scala %}
@ some(Salary(1.0f)) ?|? some(Salary(2.0f))
res8: Ordering = LT
{% endcodeblock %}  

You can now also use `sorted` on a collection equally easily. Using `Order[A].toScalaOrdering` we can get back an object of type `scala.math.Ordering` which we can use to sort collections. 

{% codeblock lang:scala %}
@ implicit val salaryOrdering = Order[Salary].toScalaOrdering
salaryOrdering: math.Ordering[Salary] = scalaz.Order$$anon$1@5b8a5d7c
@ List(Salary(2.0f), Salary(1.0f), Salary(0.0f)).sorted
res10: List[Salary] = List(Salary(0.0F), Salary(1.0F), Salary(2.0F))
{% endcodeblock %}