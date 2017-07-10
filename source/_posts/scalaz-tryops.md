---
title: Scalaz TryOps
date: 2017-07-06 12:41:25
tags:
  - scalaz
  - scala
  - functional programming
---

In this post we'll look at [`TryOps`](https://github.com/scalaz/scalaz/blob/c0e4b531847348e1fd533c7d3605fe69320dde91/core/src/main/scala/scalaz/syntax/std/TryOps.scala) and the goodies it provides to work with [`scala.util.Try`](http://www.scala-lang.org/api/2.9.3/scala/util/Try.html). To recap, here's what `Try` does:  

*The Try type represents a computation that may either result in an exception, or return a successfully computed value. It's similar to, but semantically different from the scala.util.Either type.*  

### Converting to a Disjunction  

{% codeblock lang:scala %}
@ import scalaz._
import scalaz._
@ import Scalaz._
import Scalaz._
@ import scala.util.Try
import scala.util.Try
// an operation that may potentially throw an exception
@ val t1 = Try { "1".toInt }
t1: Try[Int] = Success(1)
// converting to a Scalaz disjunction
@ val disjunction = t1 toDisjunction
disjunction: Throwable \/ Int = \/-(1)
{% endcodeblock %}  

The result of a `Try` is either a `Success` or  a `Failure`. This can very easily be translated to a Scalaz disjunction. A `Success` produces a right disjunction whereas a `Failure` produces a left disjunction.  

### Converting to a Validation  

{% codeblock lang:scala %}
@ val validation = t1 toValidation
validation: Validation[Throwable, Int] = Success(1)
{% endcodeblock %}  

Similarly, if this `Try` were a part of validating your data like checking values in a JSON object, you can convert this to a Scalaz `Validation`.  

### Converting to a ValidationNel  

{% codeblock lang:scala %}
@ val nel = t1 toValidationNel
nel: ValidationNel[Throwable, Int] = Success(1)
{% endcodeblock %}  

`ValidationNel` is useful for accumulating errors. We'll cover all of this in coming posts.   

## Conclusion  

This brings us to the end of the post on `TryOps`. In coming posts we'll look at `Validation` type which lets us represent, as you might have guessed, the result of validating an input. Similarly, if we want to accumulate all the results of validating inputs, we use `ValidationNel`. Both of these are subjects of coming posts.
