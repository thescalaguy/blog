---
title: Scalaz Equals
tags:
  - scalaz
  - scala
  - functional programming
date: 2017-06-28 20:33:52
---


In this post we'll finally start using Scalaz. We'll look at how to get Scalaz using `sbt` and look at how it provides us with type-safe equality checking.  

## Getting Scalaz  

Add the following line to your `build.sbt` file:  

{% codeblock lang:scala %}
libraryDependencies += "org.scalaz" %% "scalaz-core" % "7.2.14"
{% endcodeblock %}  

<hr> 
##### NOTE: 
I highly recommend [Ammonite Scala REPL](http://ammonite.io/). It provides a lot of improvements over the standard REPL like being able to import library dependencies straight from the console. I'll be using Ammonite REPL henceforth because it'll help me keep the examples in the REPL. However, there isn't any difference beyond how to get the dependencies.
<hr>  

## Fire Up the REPL  

No matter what your preferred REPL is, let's get started.  

### Standard REPL  

Start the REPL by executing `sbt console`. Then, execute the following:

{% codeblock lang:scala %}
scala> import scalaz._
import scalaz._

scala> import Scalaz._
import Scalaz._
{% endcodeblock %}  

### Ammonite  

Start the REPL by executing the `amm` command. Then, execute the following:

{% codeblock lang:scala %}
@ import $ivy.`org.scalaz::scalaz-core:7.2.14`
import $ivy.$
@ import scalaz._
import scalaz._
@ import Scalaz._
import Scalaz._
{% endcodeblock %}

## Equality Checking  

We can compare two values in Scala using `==` (double equals). The issue, however, is that it will let us compare unrelated types like a string with an integer and such a comparison would always yield false.  

{% codeblock lang:scala %}
@ 1 == "1"
res3: Boolean = false
{% endcodeblock %}  

The `==` operator that Scala provides is a null-safe comparison operator and not type-safe. What if we want type-safety, too? This is where Scalaz's `===` (triple equals) comes in. It'll complain when you try to compare unrelated types.  

{% codeblock lang:scala %}
@ 1 === "1"
cmd4.sc:1: type mismatch;
 found   : String("1")
 required: Int
val res4 = 1 === "1"
                 ^
Compilation Failed
{% endcodeblock %}  

Similarly, we can check for inequalities. The Scala operator `!=` is null-safe but not type-safe.  

{% codeblock lang:scala %}
@ 1 != "1"
res4: Boolean = true
{% endcodeblock %}  

Here's the Scalaz way to check for inequality using `=/=` operator which is both type-safe and null-safe:  

{% codeblock lang:scala %}
@ 1 =/= "1"
cmd5.sc:1: type mismatch;
 found   : String("1")
 required: Int
val res5 = 1 =/= "1"
                 ^
Compilation Failed
{% endcodeblock %}  

## Under the Hoods  

As always, there are type classes at play here. There is an [`Equal`](https://github.com/scalaz/scalaz/blob/fabab8f699d56279d6f2cc28d02cc2b768e314d7/core/src/main/scala/scalaz/Equal.scala) trait which provides an `equal` to check if the two values are equal and of the same type.  

{% codeblock lang:scala %}
def equal(a1: F, a2: F): Boolean
{% endcodeblock %}  

Since all this magic is done using type classes, how about we put it to use and write code to compare two `Person` objects?  

{% codeblock lang:scala %}
@ case class Person(id: Int, name: String)
defined class Person
@ implicit object PersonEquals extends Equal[Person] {
    // we are using Scalaz === internally
    def equal(a1: Person, a2: Person): Boolean = a1.id === a2.id && a1.name === a2.name 
  }
defined object PersonEquals
@ Person(1, "John") === Person(2, "Jane")
res7: Boolean = false
@ Person(1, "John") =/= Person(2, "Jane")
res8: Boolean = true
{% endcodeblock %}