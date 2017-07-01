---
title: Scalaz Show
date: 2017-07-01 11:01:40
tags:
  - scalaz
  - scala
  - functional programming
---

In this post we'll look at Scalaz [`Show`](https://github.com/scalaz/scalaz/blob/fabab8f699d56279d6f2cc28d02cc2b768e314d7/core/src/main/scala/scalaz/Show.scala). The only purpose of this trait is to provide a `String` representation for its subtypes.  

## A Simple Example

{% codeblock lang:scala %}
@ import scalaz._
import scalaz._
@ import Scalaz._
import Scalaz._
@ 3.show
res2: Cord = Cord(1 [3])
@ 3.println
3
{% endcodeblock %}  

As always, there are implicits defined for `Int`, `Float`, etc. Calling `println` returns a `String` whereas calling `show` returns a [`Cord`](https://github.com/scalaz/scalaz/blob/fabab8f699d56279d6f2cc28d02cc2b768e314d7/core/src/main/scala/scalaz/Cord.scala). Going over the source code for `Cord`: 

*A Cord is a purely functional data structure for efficiently storing and manipulating Strings that are potentially very long.*  
  

## Why Use Show?  

A reasonable question to ask is why use `Show` when we have a `toString` which produces a `String` representation of objects. The answer is that `toSring` doesn't always produce a useful representation.  

{% codeblock lang:scala %}
@ println(new Thread())
Thread[Thread-185,5,main]
{% endcodeblock %}  

## Showing a Thread  

So, let's create a `Show` for `Thread`.  

{% codeblock lang:scala %}
@ implicit object ThreadShowable extends Show[Thread] {
    override def shows(t: Thread): String = s"Thread Id=${t.getId} name=${t.getName}"
  }
defined object ThreadShowable
@ val t = new Thread()
t: Thread = Thread[Thread-322,5,main]
// Scala way
@ println(t)
Thread[Thread-322,5,main]
// Scalaz way
@ t.println
Thread Id=352 name=Thread-322
{% endcodeblock %}  

## Conclusion  

`Show` is probably not that interesting and probably exists because there is `Show` class in Haskell<sup>[[1]](https://www.haskell.org/tutorial/stdclasses.html)</sup>:  

*The instances of class Show are those types that can be converted to character strings (typically for I/O)*