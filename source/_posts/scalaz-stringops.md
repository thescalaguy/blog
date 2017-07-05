---
title: Scalaz StringOps
date: 2017-07-05 14:20:50
tags:
  - scalaz
  - scala
  - functional programming
---

In this post we'll look at [`StringOps`](https://github.com/scalaz/scalaz/blob/c0e4b531847348e1fd533c7d3605fe69320dde91/core/src/main/scala/scalaz/syntax/std/StringOps.scala) and the goodies it provides to work with `String`s. Like we did last time, we'll jump straight into examples.  

### Plural  

{% codeblock lang:scala %}
@ import scalaz._
import scalaz._
@ import Scalaz._
import Scalaz._
// keep the doctor away
@ "apple" plural 2
res2: String = "apples"
// oops!
@ "dress" plural 3
res3: String = "dresss"
// singular
@ "apple" plural 1
res4: String = "apple"
// is it Friday, yet?
@ "day" plural 3
res5: String = "days"
{% endcodeblock %}  

Scalaz provides a convenient `plural` method on `String` to get its plural form. Going over the docs for `plural`:  

*Returns the same String value if the given value is 1 otherwise pluralises this String by appending an "s" unless this String ends with "y" and not one of ["ay", "ey", "iy", "oy", "uy"] in which case the 'y' character is chopped and "ies" is appended.*  

This explains why the plural of "dress" was "dresss" with 3 "s"; `plural` simply appended an "s". Nonetheless, this is a convenient method.  

### Non-Empty List  

{% codeblock lang:scala %}
@ "list" charsNel
res6: Option[NonEmptyList[Char]] = Some(NonEmpty[l,i,s,t])
@ "" charsNel
res7: Option[NonEmptyList[Char]] = None
{% endcodeblock %}  

`charsNel` converts the `String` into an `Option[NonEmptyList[Char]]`. This method is useful if the string represents a sequence of actions to be taken as characters. For example, "OCOC" could mean "Open, Close, Open, Close" or something and `charsNel` would convert this into an `Option` of `NonEmptyList[Char]`. You can then iterate over the characters and take whatever action you want to. 

### Parsing  

Scalaz provides convenient methods for parsing a `String` into `Boolean`, `Int`, etc. The advantage of using these over standard Scala parsing methods is that these methods return a `Validation`. You can then fold the `Validation` object to see whether the value was successfully parsed or resulted in an error. This is good for functional programming as you don't have to catch exceptions. We'll first look at the Scala way and then the Scalaz way.

{% codeblock lang:scala %}
// Scala way
@ "1".toInt
res8: Int = 1
@ "x".toInt
java.lang.NumberFormatException: For input string: "x"
...
// Scalaz way
@ "1".parseInt.fold(err => "failed parse".println, num => num.println)
1
@ "x".parseInt.fold(err => "failed parse".println, num => num.println)
"failed parse"
{% endcodeblock %}  

It's also possible to parse values to `BigDecimal` and `BigInt`.

{% codeblock lang:scala %}
@ "3.14159265358979323846".parseBigDecimal.fold(err => "failed parse".println, num => num.println)
3.14159265358979323846
{% endcodeblock %}  

## Conclusion  

This covers the convenience methods Scalaz provides for dealing with `String`s. The `parseXXX` methods are the most useful as they avoid having to deal with an exception.