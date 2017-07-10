---
title: Scalaz BooleanOps
date: 2017-07-10 15:11:40
tags:
  - scalaz
  - scala
  - functional programming
---
  
In this post we'll look at [`BooleanOps`](https://github.com/scalaz/scalaz/blob/c0e4b531847348e1fd533c7d3605fe69320dde91/core/src/main/scala/scalaz/syntax/std/BooleanOps.scala) and the goodies it provides to work with `Boolean`s. As always, we'll go straight to examples.  

### Unless 

{% codeblock lang:scala %}
@ import scalaz._
import scalaz._
@ import Scalaz._
import Scalaz._
@ val t = true
t: Boolean = true
@ val f = false
f: Boolean = false
// executes the given side-effect if this boolean value is false
@ t unless "this won't print".println

@ f unless "this will print".println
"this will print"
{% endcodeblock %}  

As the comment for `unless` states:  

*Executes the given side-effect if this boolean value is false.*  

A mnemonic to remember the working is: "Unless the value is true, execute the side-effecting function".  

### When  

{% codeblock lang:scala %}
// executes the given side-effect if this boolean value is true
@ t when "this will print".println
"this will print"

@ f when "this won't print".println
{% endcodeblock %}  

The "opposite" of `unless` is `when` which executes the function when the value is true. As the comment for `when` states:  

*Executes the given side-effect if this boolean value is true.*   

A mnemonic to remember the working is: "When the value is true, execute the side-effecting function".  

### Folding a Boolean  

{% codeblock lang:scala %}
@ t fold[String]("this will be returned when true", "this will be returned when false")
res8: String = "this will be returned when true"
@ f fold[String]("this will be returned when true", "this will be returned when false")
res9: String = "this will be returned when false"
{% endcodeblock %}  

`fold` lets you decide what value to return depending on whether it is true or false.   

### Converting to an Option  

{% codeblock lang:scala %}
@ t option "this will create a Some() with this string in it"
res10: Option[String] = Some("this will create a Some() with this string in it")
@ f option "this will result in a None"
res11: Option[String] = None
{% endcodeblock %}  

`option` lets us convert a `Boolean` into an `Option` in a type-safe manner. A true results in a `Some` containing the value passed to `option` whereas a false results in an `Option` of whatever the type of the argument is.  

### Ternary Operator  

{% codeblock lang:scala %}
@ t ? "true" | "false"
res13: String = "true"
@ f ? "true" | "false"
res14: String = "false"
{% endcodeblock %}  

Scalaz also provides a ternary operator to work with `Boolean`s. The ternary operator is actually a combination of `?` and `|`. `?` is the conditional operator that results in the creation of an object of `Conditional` and `|` is a method of that object.  

### Miscellaneous   

{% codeblock lang:scala %}
@ t ?? List(1, 2, 3)
res15: List[Int] = List(1, 2, 3)
@ f ?? List(1, 2, 3)
res16: List[Int] = List()
{% endcodeblock %}  

`??` returns the given argument if the value is true, otherwise, the zero element for the type of the given argument. In our case, the "zero" element for `List` is an empty `List`.  

{% codeblock lang:scala %}
@ t !? List(1, 2, 3)
res17: List[Int] = List()
@ f !? List(1, 2, 3)
res18: List[Int] = List(1, 2, 3)
{% endcodeblock %}   

`!?` is the opposite of `??` and returns the argument if the value is false or the zero element otherwise.  

## Conclusion  

This brings us to the end of our post on `BooleanOps`. There's a lot more functions provided but I've chosen to cover those which I feel will be the most useful.