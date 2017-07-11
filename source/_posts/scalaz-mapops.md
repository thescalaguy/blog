---
title: Scalaz MapOps
date: 2017-07-11 12:17:09
tags:
  - scalaz
  - scala
  - functional programming
---

In this post we'll look at [`MapOps`](https://github.com/scalaz/scalaz/blob/fabab8f699d56279d6f2cc28d02cc2b768e314d7/core/src/main/scala/scalaz/syntax/std/MapOps.scala) and the goodies it provides to work with `Map`s. Examples galore!  

### Altering a Map

{% codeblock lang:scala %}
@ import scalaz._
import scalaz._
@ import Scalaz._
import Scalaz._

// create two maps
@ val m1 = Map("a" -> 1, "b" -> 2)
m1: Map[String, Int] = Map("a" -> 1, "b" -> 2)
@ val m2 = Map("b" -> 2, "c" -> 3)
m2: Map[String, Int] = Map("b" -> 2, "c" -> 3)

// alter an existing key
@ m1.alter("b") { maybeValue => maybeValue some { v =>  some(v + 1) } none { some(0) } }
res4: Map[String, Int] = Map("a" -> 1, "b" -> 3)
{% endcodeblock %}  

`alter` lets you change the values associated with keys. In the example, we're altering the value associated with key `b`. Since there may or may not be a value associated with `b`, we get an `Option` which we've named as `maybeValue`. We then use the `some { ... } none { ... }` construct to either add 1 in case there's a value or initialize the key with 0. Also, since we need to return `Option`s, we use `some` and `none` again.

{% codeblock lang:scala %}
@ m1.alter("d") { maybeValue => maybeValue some { v => some(v + 1) } none { some(0) } }
res5: Map[String, Int] = Map("a" -> 1, "b" -> 2, "d" -> 0)
{% endcodeblock %}  

If the key does not exist, it will be added to the map.  

### Intersection  

{% codeblock lang:scala %}
@ m1.intersectWith(m2) { _ + _ }
res6: Map[String, Int] = Map("b" -> 4)
@ m1.intersectWith(m2)((v1, v2) => v1 + v2)
res7: Map[String, Int] = Map("b" -> 4)
{% endcodeblock %}  

`intersectWith` lets you calculate the intersection of two maps. It expects a function that will accept values `v1` and `v2` from both the maps and return a new value which will become the value associated with the key in the new map. If you want to analyze the key before returning the value, you can use `intersectWithKey` which expects a function `(k, v1, v2) => { ... }`.  

### Mapping Keys  

{% codeblock lang:scala %}
@ m1 mapKeys { _.toUpperCase }
res8: Map[String, Int] = Map("A" -> 1, "B" -> 2)
{% endcodeblock %}  

`mapKeys` lets you change the keys without changing the values associated with them. Here, we're converting them to uppercase.   

### Union  

{% codeblock lang:scala %}
@ m1.unionWith(m2){ _ + _ }
res9: Map[String, Int] = Map("a" -> 1, "b" -> 4, "c" -> 3)
@ m1.unionWith(m2)((v1, v2) => v1 + v2)
res10: Map[String, Int] = Map("a" -> 1, "b" -> 4, "c" -> 3)
{% endcodeblock %}  

`unionWith` lets you calculate the union of two maps and expects a function that will accept values `v1` and `v2` from both maps and return a new value which will become the value associated with the key in the new map. Similarly, there's `unionWithKey` which will pass the key as the first argument to the function.  

### Inserting a Value

{% codeblock lang:scala %}
@ m1.insertWith("a", 99) { _ + _ }
res11: Map[String, Int] = Map("a" -> 100, "b" -> 2)
@ m1.insertWith("a", 99)((v1, v2) => v1 + v2)
res12: Map[String, Int] = Map("a" -> 100, "b" -> 2)
@ m1.insertWith("z", 100) { _ + _ }
res13: Map[String, Int] = Map("a" -> 1, "b" -> 2, "z" -> 100)
{% endcodeblock %}  

`insertWith` lets you insert a new key-value pair into the map. It also expects a function which will take values `v1` and `v2` as arguments to deal with cases where the key you're trying to add already exists.  

## Conclusion  

This brings us to the end of the post on `MapOps`. These extra functions make it very easy to work with an immutable `Map` by providing extra functionality not present in the Scala core library.