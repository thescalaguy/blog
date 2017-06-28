---
title: Scalaz Under the Hoods
date: 2017-06-24 15:52:04
tags: 
  - scalaz
  - scala 
  - functional programming
---

A lot of what Scalaz does is made possible by using ad-hoc polymorphism, traits, and implicits. I'll explain how this works by borrowing from [Nick Partridge's talk on Scalaz](http://bit.ly/c2eTVR).  

## Motivating Example  

Let's begin with a simple `sum` function that adds together all the elements in a `List[Int]`.

{% codeblock lang:scala %}
scala> def sum(xs: List[Int]): Int = xs.foldLeft(0)( _ + _ )
defined function sum
{% endcodeblock %}  

The `sum` function above works only with `List[Int]`. If we want to sum together a `List[Double]`, `List[Float]`, or a `List[String]`, we'd need a new implementation of `sum`. Our goal is to make `sum` function general so it would work with any of these.

## Step 1 - Monoid  

The first step towards generalizing `sum` is by using a monoid. A monoid is an algebraic structure with a single associative binary operation and an identity element.<sup>[[1]](https://en.wikipedia.org/wiki/Monoid)</sup>. Since we are working with a `List[Int]`, let's create an `IntMonoid`.  

{% codeblock lang:scala %}
scala> object IntMonoid {
    def mempty: Int = 0
    def mappend(a: Int, b: Int): Int = a + b
  }
defined object IntMonoid

scala> def sum(xs: List[Int]) = xs.foldLeft(IntMonoid.mempty)(IntMonoid.mappend)
defined function sum
{% endcodeblock %} 

`mempty` is the identity or the zero value, and  `mappend` is the binary operation which produces another `Int` i.e. another value in the set. These names come from Haskell<sup>[[2]](https://en.wikibooks.org/wiki/Haskell/Monoids)</sup>. 

## Step 2 - Generalizing the Monoid  

Next, we'll generalize the monoid by creating a `Monoid[A]` so that `IntMonoid` is just a monoid on `Int`.

{% codeblock lang:scala %}
scala> trait Monoid[A] {
    def mempty: A
    def mappend(a: A, b: A): A
  }
defined trait Monoid

scala> object IntMonoid extends Monoid[Int] {
    def mempty: Int = 0
    def mappend(a: Int, b: Int) = a + b
  }
defined object IntMonoid

scala> def sum[A](xs: List[A], m: Monoid[A]): A = xs.foldLeft(m.mempty)(m.mappend)
defined function sum

scala> sum(List(1, 2, 3), IntMonoid)
res3: Int = 6
{% endcodeblock %}  

What we've done is created a general-purpose `sum` function whose working depends upon which monoid is passed to it. Now we can very easily sum a `List[String]` or a `List[Double]` by adding a corresponding monoid.  

## Step 3 - Make the Monoid Implicit  

Next, we'll make the monoid an implicit parameter to our `sum` function. We'll also package our `IntMonoid` into a `Monoid` companion object and make it implicit. The reason for doing this is how Scala compiler resolves implicit values; it'll look for implicit values in its scope. So, we bring `IntMonoid` within scope by by importing from the `Monoid` companion object.

{% codeblock lang:scala %}
scala> trait Monoid[A] {
    def mempty: A
    def mappend(a: A, b: A): A
  }
defined trait Monoid

scala> object Monoid {
    implicit object IntMonoid extends Monoid[Int] {
      def mempty: Int = 0
      def mappend(a: Int, b: Int) = a + b
    }
  }
defined object Monoid

scala> import Monoid._
import Monoid._

scala> def sum[A](xs: List[A])(implicit m: Monoid[A]): A = xs.foldLeft(m.mempty)(m.mappend)
defined function sum

scala> sum(List(1, 2, 3))
res4: Int = 6
{% endcodeblock %}  

So, what we've done is create a general-purpose `sum` function that works as long as there is a corresponding implicit monoid within scope. This is made possible by using ad-hoc polymorphism. I'll cover ad-hoc polymorphism briefly in this post and defer providing a detailed explanation for a later post.  

## Ad-hoc Polymorphism  

Ad-hoc polymorphism is a type of polymorphism in which polymorphic functions are invoked depending on the different argument types. One way to implement ad-hoc polymorphism that we already know about is function overloading where a different "version" of the function is invoked depending on the type of arguments. This is what we've done in the post where there is only a single function but an implementation is provided for different types. In other words, `sum` only knows how to be invoked but the behavior is provided by monoids. Another way to implement ad-hoc polymorphism is coercion where the argument is converted into a type which the function expects.  

So, by using ad-hoc polymorphism, Scalaz is able to provide general-purpose functions over existing types. Ad-hoc polymorphism is flexible in the sense that it lets you extend even those classes for which you do not have access to the source code i.e. classes from other libraries, etc.