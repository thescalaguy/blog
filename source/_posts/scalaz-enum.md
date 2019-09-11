---
title: Scalaz Enum
date: 2017-07-03 19:50:05
tags:
  - scalaz
  - scala
  - functional programming
---

In this post we'll look at how to create enum using Scalaz. However, we'll first look at how to create enum using standard Scala library.  

## Enum - Scala Way  

*An enum type is a special data type that enables for a variable to be a set of predefined constants. The variable must be equal to one of the values that have been predefined for it. Common examples include compass directions (values of NORTH, SOUTH, EAST, and WEST) and the days of the week.*<sup>[[1]](https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html)</sup>   

There are a couple of ways in which you can create enum in Scala.  

### Using [`scala.Enumeration`](http://www.scala-lang.org/api/current/scala/Enumeration.html)  

*Defines a finite set of values specific to the enumeration. Typically these values enumerate all possible forms something can take and provide a lightweight alternative to case classes.*  

Let's see an example:  

{% codeblock lang:scala %}
@ object Direction extends Enumeration {
    type Direction = Value
    val NORTH, EAST, WEST, SOUTH = Value
  }
defined object Direction
@ import Direction._
import Direction._
@ Direction withName "NORTH"
res2: Value = NORTH
@ Direction.values foreach println
NORTH
EAST
WEST
SOUTH
{% endcodeblock %}    

Advantages:  

<li> Lightweight. </li>
<li> Easy to use. </li>
<li> Iterable. </li>  


Disadvantages:

<li> They only exist as unique values. You cannot add behavior to them. </li>

### Using Case Objects  

Enum can also be created using `sealed trait`. A sealed trait requires all the classes or traits extending it be in the same file in which it is defined.

{% codeblock lang:scala %}
@ object Direction {
    sealed trait Direction
    case object NORTH extends Direction
    case object EAST extends Direction
    case object WEST extends Direction
    case object SOUTH extends Direction
  }
defined object Direction
@ import Direction._
import Direction._
{% endcodeblock %}  

Advantages:  

<li> Each value is an instance of its own type and it's possible to add behavior to them. </li>  

Disadvantages:  

<li> A case object generates a lot more code than `Enumeration`. </li>  
<li> Not iterable. </li>  

## Enum - Scalaz Way  

Scalaz allows creating enum using [`Enum`](https://github.com/scalaz/scalaz/blob/fabab8f699d56279d6f2cc28d02cc2b768e314d7/core/src/main/scala/scalaz/Enum.scala) trait. Scalaz enums are more powerful than the standard Scala enums. Let's see some examples of creating Scalaz enum<sup>[[2]](http://eed3si9n.com/learning-scalaz/Enum.html)</sup>:  

{% codeblock lang:scala %}
@ import scalaz._
import scalaz._
@ import Scalaz._
import Scalaz._
// as a List
@ 'a' |-> 'z'
res3: List[Char] = List(
  'a',
  'b',
  'c',
  'd',
  'e',
  'f',
  ...
// as a Scalaz stream
@ 'a' |=> 'z'
res4: EphemeralStream[Char] = scalaz.EphemeralStream$$anon$5@2fb4925c
{% endcodeblock %}

Scalaz also provides us with `succ` and `pred` to get the successor and predecessor.

{% codeblock lang:scala %}
@ 'B'.succ
res6: Char = 'C'
@ 'B'.pred
res7: Char = 'A'
{% endcodeblock %}  

It's also possible to easily step through the enumeration.  

{% codeblock lang:scala %}
// 3 steps forwards
@ 'B' -+- 3
res8: Char = 'E'
// 3 steps backwards
@ 'B' --- 3
res9: Char = '?'
{% endcodeblock %}  

You can also get min and max values for an enumeration.  

{% codeblock lang:scala %}
@ implicitly[Enum[Int]].min
res10: Option[Int] = Some(-2147483648)
@ implicitly[Enum[Int]].max
res11: Option[Int] = Some(2147483647)
{% endcodeblock %}

Let's create our own enumeration using Scalaz `Enum`.

{% codeblock lang:scala %}
@ import scalaz._
import scalaz._
@ import Scalaz._
import Scalaz._
@ object Size {
    case class Size(val size: Int, val name: String)
    val SMALL = Size(0, "SMALL")
    val MEDIUM = Size(1, "MEDIUM")
    val LARGE = Size(2, "LARGE")

    implicit object SizeEnum extends Enum[Size] with Show[Size] {
      def order(s1: Size, s2: Size): Ordering = (s1.size compare s2.size) match {
          case -1 => Ordering.LT
          case 0 => Ordering.EQ
          case 1 => Ordering.GT
        }

        def succ(s: Size): Size = s match {
          case SMALL => MEDIUM
          case MEDIUM => LARGE
          case LARGE => SMALL
        }

        def pred(s: Size): Size = s match {
          case SMALL => LARGE
          case MEDIUM => SMALL
          case LARGE => MEDIUM
        }

        def zero: Size = SMALL

        override def shows(s: Size) = s.name

        override def max = Some(LARGE)

        override def min = Some(SMALL)
    }
  }
defined object Size
@ import Size._
import Size._
// step through
@ SMALL -+- 1
res4: Size = Size(1, "MEDIUM")
// as a list
@ SMALL |-> LARGE
res5: List[Size] = List(Size(0, "SMALL"), Size(1, "MEDIUM"), Size(2, "LARGE"))
// show
@ SMALL.println
SMALL
// max
@ implicitly[Enum[Size]].max
res6: Option[Size] = Some(Size(2, "LARGE"))
// min
@ implicitly[Enum[Size]].min
res7: Option[Size] = Some(Size(0, "SMALL"))
{% endcodeblock %}  

Notice that since we're using `implicitly[]`, we don't have to provide the exact name of the typeclass extending `Enum[Size]`, the compiler will figure it out from the implicits within scope.

## Conclusion  

Although Scalaz `Enum` may seem like a long-winded way to do something trivial, they offer you a lot more flexibility and brevity over vanilla Scala enum. However, I believe that the full force of Scalaz `Enum` is needed only when you have ordered elements like size. For cases like compass directions, Scala enum are better-suited.
