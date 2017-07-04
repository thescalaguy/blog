---
title: Scalaz OptionOps
date: 2017-07-04 16:20:10
tags:
  - scalaz
  - scala
  - functional programming
---

In this post we'll look at [`OptionOps`](https://github.com/scalaz/scalaz/blob/c0e4b531847348e1fd533c7d3605fe69320dde91/core/src/main/scala/scalaz/syntax/std/OptionOps.scala) and the goodies it provides to work with `Option`s. We'll jump straight into examples.  

### Creating Options

{% codeblock lang:scala %}
@ import scalaz._
import scalaz._
@ import Scalaz._
import Scalaz._
// Some
@ some(13)
res2: Option[Int] = Some(13)
// Calling some on a value
// Notice something about res variable? ;)
@ 13.some
res9: Option[Int] = Some(13)
// None
@ none
res3: Option[Nothing] = None
@ none[Int]
res4: Option[Int] = None
{% endcodeblock %}  

### Extracting Values  

{% codeblock lang:scala %}
// using unary ~ operator
@ ~ res2
res5: Int = 13
// using some / none
@ res2 some { _ * 2 } none { 0 }
res6: Int = 26
// using | operator
@ res2 | 0
res7: Int = 13
// extracting value from none
@ ~ res4
res8: Int = 0
{% endcodeblock %}  

The unary `~` extracts the value from the `Option` or returns the zero value for that type. For example, in case of `Int` it'll return the value in the `Option` or return 0. The `some{ .. } none { .. }` construct lets you specify what value to return in case of `Some` and `None`. I am multiplying the value by 2 and thus returning 26. As you can see, Scalaz provides with more expressive and type-safe way to create and extract values from `Option`.   

Notice that the type of all the resulting variables is `Option` instead of `Some` or `None`. This is more type-safe than using `None` because you'd get `None.type`, and the type inferencer would allow the type to be `Option[Int]` or `Option[String]` whereas `none[Int]` is guaranteed to be an `Option[Int]`. The type inferencer will enforce this. 

### Miscellaneous  

{% codeblock lang:scala %}
// we'll have fun with these values
@ val o = some("option")
o: Option[String] = Some("option")
@ val n = none[String]
n: Option[String] = None

// None to right disjunction
@ n toRightDisjunction "onTheLeft"
res10: String \/ String = -\/("onTheLeft")
// Folding the disjunction prints the value on the left
@ res10 fold(l => s"Left: $l".println, r => s"Right: $r".println)
"Left: onTheLeft"

// Some to right disjunction
@ o toRightDisjunction "onTheLeft"
res11: String \/ String = \/-("option")
// Folding the disjunction prints the value on the right
@ res11 fold(l => s"Left: $l".println, r => s"Right: $r".println)
"Right: option"

// default values, similar to unary ~
@ n.orZero
res12: String = ""

// ternary operator
@ o ? "some" | "none"
res13: String = "some"
{% endcodeblock %}  

Scalaz provides with a disjunction which is a replacement for Scala `Either`. I've only introduced disjunction here and will cover it later in a post of its own. Trying to convert a `None` to a right disjunction doesn't work. We get back a left disjunction as indicated by `-\/`. Similarly, converting a `Some` to right disjunction works, as indicated by `\/-`. Scalaz also provides a ternary operator to work with `Option`s. 

## Conclusion  

This sums up, non-exhaustively, how we can use the convenience methods that Scalaz provides on top of `Option`s that lets us code in more expressive and type-safe way. 