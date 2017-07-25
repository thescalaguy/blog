---
title: Scalaz Either
date: 2017-07-17 14:15:10
tags:
  - scalaz
  - scala
  - functional programming
---

In this post we'll look at [Scalaz `Either`](https://github.com/scalaz/scalaz/blob/series/7.3.x/core/src/main/scala/scalaz/Either.scala). This is Scalaz's version of the standard [Scala `Either`](http://www.scala-lang.org/api/2.9.3/scala/Either.html). Before we look at Scalaz `Either`, we'll look at Scala `Either`.

## Scala Either  

Let's begin by going over the docs:<sup>[[1]](http://www.scala-lang.org/api/2.12.0/scala/util/Either.html)</sup>  

*Represents a value of one of two possible types (a disjoint union.) Instances of Either are either an instance of Left or Right.  
...    
Convention dictates that Left is used for failure and Right is used for success.*  

With the definition out of the way, let's look at some code. The example is modeled around the code in the official Scala `Either` docs.  

### Creating an Either

{% codeblock lang:scala %}
@ def parseInt(str: String): Either[String, Int] = {
      try {
          Right(str toInt)
      } catch {
          case e: Exception => Left(e getMessage)
      }
  }
defined function parseInt
{% endcodeblock %}

Next, let's create a case each of success and failure.  

{% codeblock lang:scala %}
@ val success = parseInt("2")
success: Either[String, Int] = Right(2)
@ val failure = parseInt("apple")
failure: Either[String, Int] = Left("For input string: \"apple\"")
{% endcodeblock %}  

### Using a `for` Comprehension

Scala `Either` is not a monad and so you cannot use it in a `for` comprehensions.  

{% codeblock lang:scala %}
@ for {
      n <- success
  } yield n
res3: Either[String, Int] = Right(2)
{% endcodeblock %} 

<hr>
#### NOTE:
Previously, Scala `Either` was not a monad so it couldn't be used in `for` comprehensions. Now, it is a monad and can be used in `for` comprehensions.
<hr> 

## Scalaz Either  

A valid question to ask is why would one use Scalaz `Either` when Scala `Either` is a monad. The answer is that Scalaz `Either` is a lot more convenient and powerful compared to Scala `Either`. Let's begin by refactoring `parseInt` to return a Scalaz `Either`.  

### Creating an Either

{% codeblock lang:scala %}
@ import scalaz._
import scalaz._
@ import Scalaz._
import Scalaz._
@ def parseInt(str: String): String \/ Int = {
      import scala.util.{Try, Success, Failure}
      Try { str toInt } match {
          case Success(n) => n.right[String]
          case Failure(e) => e.getMessage.left[Int]
      }
  }
defined function parseInt
{% endcodeblock %}  

Next, let's create a case each of success and failure.

{% codeblock lang:scala %}
@ val success = parseInt("2")
success: String \/ Int = \/-(2)
@ val failure = parseInt("apple")
failure: String \/ Int = -\/("For input string: \"apple\"")
{% endcodeblock %}

The return type of our function is indicated by `String \/ Int`. This means we may return a `String` on the left in case of failure and an `Int` on the right in case of success. We create right or left projections by calling `right` or `left`, respectively, and mentioning the type of the value that will be on the other side. For example, we call `right[String]` because the left side is a `String`. The right projection is indicated by `\/-` and left projection is indicated by `-\/`.  

### Using a `for` Comprehension

{% codeblock lang:scala %}
@ for {
      n <- success
  } yield n
res12: String \/ Int = \/-(2)
{% endcodeblock %}

Because Scalaz `Either` is also a monad, it can be used in a `for` comprehension.  

### Checking Left or Right

{% codeblock lang:scala %}
@ success isRight
res13: Boolean = true
@ failure isLeft
res14: Boolean = true
{% endcodeblock %}  

Akin to Scala `Either`, Scalaz `Either` also lets you check for left or right by calling `isLeft` or `isRight`, respectively.  

### Ternary Operator

{% codeblock lang:scala %}
@ success ? "YES" | "NO"
res15: String = "YES"
{% endcodeblock %}  

Scalaz `Either` provides you with a `getOrElse` which you can use to as a ternary operator using its symbolic representation `|`.  

### Folding an Either  

{% codeblock lang:scala %}
@ success fold(
    left => -1,
    right => right + 1
  )
res16: Int = 3
@ failure fold(
    left => -1,
    right => right + 1
  )
res17: Int = -1
{% endcodeblock %}  

Both Scala and Scalaz `Either` provide you with a `fold` method which run the first function if we have a left, or the second function if we have a right.  

### Converting to Validation

The single biggest difference between Scala `Either` and Scalaz `Either` is that Scalaz `Either` can be converted to other types like `Validation`, etc. For example, converting an `Either` to a `Validation` allows you to accumulate errors. As the code comments state:  

*`A \/ B` is also isomorphic to `Validation[A, B]`. The subtle but important difference is that `Applicative` instances for `Validation` accumulates errors ("lefts")*  

{% codeblock lang:scala %}
@ 1.right validation
res65: Validation[Nothing, Int] = Success(1)
{% endcodeblock %}  

We create a `Validation` by calling `validation` method on the `Either` instance. Depending on a left or right, we get either a `Success` or `Failure`.

## Conclusion

Scalaz `Either` and Scala `Either` are pretty similar in the latest version of Scala (2.12, as of writing). Which one you decide to use depends upon your personal preference. My preference is to use Scalaz `Either` throughout my code if I am using other Scalaz features to maintain consistency.