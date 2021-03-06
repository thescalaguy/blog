---
title: Scalaz TupleOps
date: 2017-07-05 17:15:10
tags:
  - scalaz
  - scala
  - functional programming
---

In this post we'll look at `TupleOps` and the goodies it provides to work with `Tuple`s. There's no `TupleOps.scala` file in the GitHub repo because the file is generated by [`GenerateTupleW`](https://github.com/scalaz/scalaz/blob/c0e4b531847348e1fd533c7d3605fe69320dde91/project/GenerateTupleW.scala). You can see the code for `TupleOps` if you're using an IDE like IntelliJ. There's a number of `TupleNOps` classes like `Tuple2Ops[A, B]`, etc. all the way to `Tuple12Ops[A, B]`.   

With that said, let's jump into examples. We'll look at what `Tuple2Ops` provides us. Everything is analogous to `Tuple3Ops`, etc. It's just the number of arguments to the methods that will change. 

### Folding a Tuple  

{% codeblock lang:scala %}
@ import scalaz._
import scalaz._
@ import Scalaz._
import Scalaz._
@ (1, 2) fold( (_1, _2) => _1 + _2 )
res2: Int = 3
{% endcodeblock %}  

`fold` in `Tuple2Ops` takes a function which accepts 2 arguments i.e. equal to the arity of the tuple. Here, we're folding the tuple and adding together its two `Int` elements but you can do whatever you want.  

### Converting to IndexedSeq  

{% codeblock lang:scala %}
@ (1, 2) toIndexedSeq
res3: collection.immutable.IndexedSeq[Int] = Vector(1, 2)
{% endcodeblock %}  

This one is self-explanatory. The tuple is converted to an [`IndexedSeq`](http://www.scala-lang.org/api/current/scala/collection/IndexedSeq.html).  

### Mapping Elements in a Tuple  

{% codeblock lang:scala %}
@ (1, 2) mapElements(_1 => _1 * 2, _2 => _2 * 2)
res4: (Int, Int) = (2, 4)
{% endcodeblock %}  

`mapElements` in `Tuple2Ops` takes 2 functions as arguments, 1 for each element of the tuple. The functions are then applied to their corresponding elements and the result is returned as a `Tuple`. In the example above, we're just multiplying both the elements by 2.   

This is different from the `map` from the standard library. The `map` from the standard library only operates on the last element of the tuple.  

{% codeblock lang:scala %}
@ (1, 2) map (_ * 2)
res5: (Int, Int) = (1, 4)
{% endcodeblock %}

## Conclusion  

That's the end of the post on `TupleOps`. `TupleOps`'s convenience methods can be used to manipulate tuples easily.