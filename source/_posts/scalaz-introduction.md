---
title: Scalaz Introduction
date: 2017-06-23 15:20:22
tags: 
  - scalaz
  - scala 
  - functional programming
---
## Motivation

I've been using Scalaz for a while now and I remember not having any guides that provide a gentle introduction. It was a lot of scouring around, reading source code, and watching tech talks that gave me the understanding that I have now. This series of posts is intended at filling that gap by providing simple, step-by-step tutorials that will help you get productive quickly without compromising on the functional programming concepts. 

## What is Scalaz? 

The documentation for Scalaz (pronounced Scala-zee or Scala-zed) states:

_Scalaz is a Scala library for functional programming._
_It provides purely functional data structures to complement those from the Scala standard library. It defines a set of foundational type classes (e.g. Functor, Monad) and corresponding instances for a large number of data structures._

In a nutshell, Scalaz aims to do three things:
<li> Provide new datatypes that are not present in the core Scala library </li>
<li> Provide new operations on existing types a.k.a. pimp the library </li>
<li> Provide general-purpose functions so that you don't have to re-write them </li> 

I'll provide a quick example of each of these without going into any details.  

## New Datatypes  

{% codeblock lang:scala %}
scala> import scalaz._
import scalaz._

scala> import Scalaz._
import Scalaz._

scala> NonEmptyList.nels(1, 2, 3)
res0: scalaz.NonEmptyList[Int] = NonEmpty[1,2,3]
{% endcodeblock %}  

Here we are creating a `NonEmptyList` which is a list that is guaranteed to have atleast one element i.e. it's never empty.

## New Operations on Existing Types  

{% codeblock lang:scala %}
scala> import scalaz._
import scalaz._

scala> import Scalaz._
import Scalaz._

scala> val o = Option(3)
o: Option[Int] = Some(3)

// Scalaz way
scala> o some { _ + 1 } none { 0 }
res0: Int = 4

// Scala way
scala> o.fold(0)( _ + 1 )
res1: Int = 4
{% endcodeblock %}

The first way of extracting value from an `Option` comes from Scalaz and is much more expressive compared to using `fold` from Scala standard library.  

## General-Purpose Functions 

{% codeblock lang:scala %}
scala> import scalaz._
import scalaz._

scala> import Scalaz._
import Scalaz._

scala> List(1, 2, 3) |+| List(4, 5, 6)
res0: List[Int] = List(1, 2, 3, 4, 5, 6)
{% endcodeblock %}

The `|+|` operator from Scalaz conveniently concatenated the two `List`s together.  