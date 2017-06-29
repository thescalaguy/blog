---
title: Scalaz Types of Polymorphism
date: 2017-06-27 12:54:44
tags: 
  - scalaz
  - scala 
  - functional programming
---

Polymorphism is a programming language feature that allows one interface to be used for a general class of actions.<sup>[[1]](http://scala.org.ua/presentations/scala-polymorphism/#/1)</sup> Scalaz makes extensive use of ad-hoc polymorphism to provide its set of goodies. In this post I'll cover ad-hoc polymorphism in detail and show you how you can implement ad-hoc polymorphism in Scala. I'll also talk about parametric, and subtype polymorphism.  

## Parametric Polymorphism  

In parametric polymorphism, the behavior of the function does not depend upon the type of the arguments passed to it. More formally<sup>[[2]](https://wiki.haskell.org/Polymorphism#Parametric_polymorphism)</sup>,  

*Parametric polymorphism refers to when the type of a value contains one or more (unconstrained) type variables, so that the value may adopt any type that results from substituting those variables with concrete types.* 

{% codeblock lang:scala %}
scala> def head[A](xs: List[A]) = xs(0)
head: [A](xs: List[A])A

scala> head(List(1, 2, 3))
res0: Int = 1

scala> head(List("a", "b", "c"))
res1: String = a
{% endcodeblock %}  

Here, the argument `xs` has an unconstrained type `A` which can be anything and yet `head` would work. Then we call `head` with lists of concrete type `Int`, and `String` resulting in `xs` being of type `List[Int]` and `List[String]`.  

## Subtype Polymorphism  

Subtype polymorphism involves inheritance. More formally<sup>[[3]](http://scala.org.ua/presentations/scala-polymorphism/#/10)</sup>, 

*Subtype polymorphism is a form of type polymorphism in which a subtype is a data-type that is related to another data-type (the super-type) by some notion of substitutability.* 

As an example<sup>[[4]](http://eed3si9n.com/learning-scalaz/polymorphism.html#Subtype+polymorphism)</sup>, consider a trait `Plus` which lets its subtype add itself with another of the same type.

{% codeblock lang:scala %}
scala> trait Plus[A] {
     |   def plus(a2: A): A
     | }
defined trait Plus

scala> def plus[A <: Plus[A]](a1: A, a2: A): A = a1.plus(a2)
plus: [A <: Plus[A]](a1: A, a2: A)A

scala> case class Currency(amount: Float, code: String) extends Plus[Currency] {
     |   override def plus(other: Currency) = Currency(this.amount + other.amount, this.code)
     | }
defined class Currency

scala> case class Kilogram(amount: Float) extends Plus[Kilogram] {
     |   override def plus(other: Kilogram) = Kilogram(this.amount + other.amount)
     | }
defined class Kilogram

scala> plus(Currency(1, "USD"), Currency(1, "USD"))
res4: Currency = Currency(2.0,USD)

scala> plus(Kilogram(1), Kilogram(1))
res5: Kilogram = Kilogram(2.0)
{% endcodeblock %}  

The function `plus[A <: Plus[A]]` will work only with those arguments that are subtype of `Plus`. This is restrictive because for this to work, the trait needs to be mixed in at the time of defining whatever concrete type `A` represents.  

## Ad-hoc Polymorphism  

In ad-hoc polymorphism the behavior of the function depends on what the type of the argument is. More formally<sup>[[5]](https://wiki.haskell.org/Polymorphism#Ad-hoc_polymorphism)</sup>, 

*Ad-hoc polymorphism refers to when a value is able to adopt any one of several types because it, or a value it uses, has been given a separate definition for each of those types.*

The simplest way to do ad-hoc polymorphism is by overloading functions. For example, having `plus(Int, Int)`, `plus(Currency, Currency)`, etc. The other ways are by using [type classes](http://scala.org.ua/presentations/scala-polymorphism/#/18), and [coercion](http://scala.org.ua/presentations/scala-polymorphism/#/15).  

### Type Classes

I'll rewrite the `Plus[A]` example using type classes:  

{% codeblock lang:scala %}
scala> case class Currency(amount: Float, code: String)
defined class Currency

scala> case class Kilogram(amount: Float)
defined class Kilogram

scala> trait Plus[A] {
     |   def plus(a:A, b: A): A
     | }
defined trait Plus

scala> object Plus {
     |   implicit object CurrencyPlus extends Plus[Currency]{
     |     override def plus(a: Currency, b: Currency): Currency = Currency(a.amount + b.amount, a.code)
     |   }
     |   implicit object KilogramPlus extends Plus[Kilogram] {
     |     override def plus(a: Kilogram, b: Kilogram): Kilogram = Kilogram(a.amount + b.amount)
     |   }
     | }
defined object Plus

scala> import Plus._
import Plus._

scala> def plus[A](a: A, b: A)(implicit p: Plus[A]) = p.plus(a, b)
plus: [A](a: A, b: A)(implicit p: Plus[A])A

scala> plus(Currency(1, "USD"), Currency(1, "USD"))
res0: Currency = Currency(2.0,USD)

scala> plus(Kilogram(1), Kilogram(1))
res1: Kilogram = Kilogram(2.0)
{% endcodeblock %}  

Relating the above implementation to the formal definition, the implicit `p` was able to adopt one of `CurrencyPlus` or `KilogramPlus` depending on what arguments were passed. Thus the behavior of `plus` is polymorphic as its behavior comes from the definition of each of the types.  

### Coercion

The other way to implement ad-hoc polymorphism is coercion. Coercion refers to implicitly converting the argument into a type that the function expects. Here's an example modeled around [scala.runtime.RichInt](https://github.com/scala/scala/blob/05016d9035ab9b1c866bd9f12fdd0491f1ea0cbb/src/library/scala/runtime/RichInt.scala):  

{% codeblock lang:scala %}
scala> class AwesomeInt(val self: Int) {
     |   def absolute: Int = math.abs(self)
     | }
defined class AwesomeInt

scala> object AwesomeInt {
     |   import scala.language.implicitConversions
     |   implicit def asAwesomeInt(x: Int): AwesomeInt = new AwesomeInt(x)
     | }
defined object AwesomeInt

scala> import AwesomeInt._
import AwesomeInt._

scala> -1.absolute
res0: Int = 1
{% endcodeblock %}  

The `absolute` method is defined in `AwesomeInt` but what we have is a plain old `Int`. This `Int` gets transformed into an `AwesomeInt` automagically because we have an implicit conversion within scope. The `Int` was coerced into an `AwesomeInt`.  

So we see how ad-hoc polymorphism allows us to extend classes to whose code we don't have access. We defined `absolute` to work with `Int` which comes from the Scala core library.  

## Higher-Kinded Types (HKT)

In the post where we generalized `sum` function, we generalized it to work with a `List`. What if we want to generalize it even further so that it can work with not just list but anything? Here's what we want to achieve:  

{% codeblock lang:scala %}
// we have this
def sum[A](xs: List[A])(implicit m: Monoid[A]): A = ???

// we want this
def sum[M[_], A](xs: M[A])(implicit m: Monoid[A], f: FoldLeft[M]): A = ???
{% endcodeblock %}

The difference between the two is that the first version, although polymorphic over types for which we have monoids defined, is still heavily tied to `List`. The second version instead will work with type that is `FoldLeft`. We'll expand upon the `sum` example<sup>[[6]](http://bit.ly/c2eTVR)</sup>. We'll need the monoids and we'll need to add a `FoldLeft` to make `sum` more generic.  

{% codeblock lang:scala %}
scala> import scala.language.higherKinds
import scala.language.higherKinds

scala> trait Monoid[A] {
     |   def mempty: A
     |   def mappend(a: A, b: A): A
     | }
defined trait Monoid

scala> trait FoldLeft[F[_]] {
     |   def foldLeft[A, B](xs: F[A], b: B, f: (B, A) => B): B
     | }
defined trait FoldLeft

scala> object FoldLeft {
     |   implicit object FoldLeftList extends FoldLeft[List] {
     |     def foldLeft[A, B](xs: List[A], b: B, f: (B, A) => B): B = xs.foldLeft(b)(f)
     |   }
     | }
defined object FoldLeft

scala> object Monoid {
     |   implicit object IntMonoid extends Monoid[Int] {
     |     def mempty: Int = 0
     |     def mappend(a: Int, b: Int) = a + b
     |   }
     | }
defined object Monoid

scala> import Monoid._
import Monoid._

scala> import FoldLeft._
import FoldLeft._

scala> def sum[M[_], T](xs: M[T])(implicit m: Monoid[T], f: FoldLeft[M]): T = f.foldLeft(xs, m.mempty, m.mappend)
sum: [M[_], T](xs: M[T])(implicit m: Monoid[T], implicit f: FoldLeft[M])T

scala> sum(List(1, 2, 3))
res0: Int = 6
{% endcodeblock %}  

So, by using HKT, we can generalize across types.   

## Pimp My Library  

You might wonder what the whole point of using so much polymorphism is. The answer is that it lets you inject new functionality into existing libraries without having access to their source code. To quote Martin Odersky<sup>[[7]](http://www.artima.com/weblogs/viewpost.jsp?thread=179766)</sup>:  

*There's a fundamental difference between your own code and libraries of other people: You can change or extend your own code, but if you want to use some other libraries you have to take them as they are ... Scala has implicit parameters and conversions. They can make existing libraries much more pleasant to deal with.*  

So, by using all the polymorphism techniques, we can implement the "pimp my library" pattern.<sup>[[8]](http://groovy-lang.org/design-patterns.html#_pimp_my_library_pattern)</sup>

*The Pimp my Library Pattern suggests an approach for extending a library that nearly does everything that you need but just needs a little more. It assumes that you do not have source code for the library of interest.*

## Conclusion

The polymorphism capabilities provided by Scala allow Scalaz to provide its set of features. By using HKT we can write code that truly generalizes across multiple types. It might seem like a long-winded way to do something so simple but as codebase gets larger and larger, the features that Scalaz provides out-of-the-box really help in eliminating boilerplate code. We haven't seen how to use Scalaz, yet. These posts serve as a foundation to understand how Scalaz does what it does.