---
title: Transducers
tags:
  - clojure
  - functional programming
date: 2018-01-06 16:18:11
---

One of the nice things that you'll come across in Clojure is transducer. In this post I'll go over what transducers are, how you can use them, how you can make one, and what transducible contexts are.  

## What are transducers?  
In simple terms, transducers are composable transformation pipelines. A transducer does not care about where the input comes from or where the output will go; it simply cares about the transformation of the data that that flows through the pipeline. Let's look at an example:  

{% codeblock %}
(let [xf (comp 
           (map inc)
           (filter even?))]
  (sequence xf (range 10)))
{% endcodeblock %}  

Here `xf` (e**x**ternal **f**unction) is our transducer which will increment every number and then will keep only the even numbers. Calling sequence functions like `map`, `filter`, etc. with single arity returns a transducer which you can then `comp`ose. The transducer doesn't know where it will be used - will it be used with a collection or with a channel? So, a transducer captures the essence of your transformation.  `sequence` is responsible for providing the input to the transducer. This is the context in which the transducer will run.  

Here's how the same thing can be done using threading macro:  

{% codeblock %}
(->> (range 10)
     (map inc)
     (filter even?))
{% endcodeblock %}  

The difference here is that the 2-arity version of `map` and `filter` will create intermediate collections while the 1-artity versions won't. Transducers are much more efficient than threading together sequence functions.  

![Chaining](/images/chain.gif)
![Transducing](/images/transduce.gif)
<center>[Source for images](https://medium.com/@roman01la/understanding-transducers-in-javascript-3500d3bd9624)</center>

## Inside a transducer  
Let's look at the 1-arity version of `map` and see what makes a transducer.  

{% codeblock %}
(defn map
  ([f]
    (fn [rf]
      (fn
        ([] (rf))
        ([result] (rf result))
        ([result input]
           (rf result (f input)))
        ([result input & inputs]
           (rf result (apply f input inputs))))))
   ...)
{% endcodeblock %}  

When you call 1-arity version of `map` you get back a transducer which, as shown above, is a function. Functions like `map`, `filter`, etc. take a collection and return a collection. Transducers, on the otherhand, take one reducing function and return another. The function returned is expected to have three arities:  
<li> 0-arity (init): This kickstarts the transformation pipeline. The only thing you do here is call the reducing function `rf`.</li>
<li> 2-arity (step): This is where you'll perform the transformation. You get the result so far and the next input. In case of `map`, you call the reducung function `rf` by applying the function `f` to the input. How the value is going to be added to the result is the job of `rf`. If you don't want to add anything to the result, just return the result as-is. You may call `rf` once, multiple times, or not at all.</li>
<li> 1-arity (end): This is called when the transducer is terminating. Here you must call `rf` exactly once and call the 1-arity version. This results in the production of the final value.</li>  

So, the general form of a transducer is this:  

{% codeblock %}
(fn [rf]
  (fn 
    ([] (rf))
    ([result] (rf result))
    ([result input] ... )))
{% endcodeblock %}  

## Using transducers

You can use a transducer in a context. There's four contexts which come out of the box — `into`, `transduce`, `sequence`, and `educe`.

### into  
The simplest way to use a transducer is to pass it to `into`. This will add your transformed elements to an already-existing collection after applying the transducer. In this example, we're simply adding a range into a vector.  

{% codeblock %}
(let [xf (comp (map inc)
               (map dec))]
  (into [] xf (range 10)))
{% endcodeblock %}  

Internally, `into` calls `transduce`.  

### transduce  
`transduce` is similar to the standard `reduce` function but it also takes an additional xform as an argument.  

{% codeblock %}
(let [xf (comp (map inc)
               (map dec))]
  (transduce xf + (range 10)))
{% endcodeblock %}

### sequence  
`sequence` lets you create a lazy sequence after applying a transducer. In contrast, `into` and `transduce` are eager.

{% codeblock %}
(let [xf (comp (map inc)
               (map dec))]
  (sequence xf (range 10)))
{% endcodeblock %}  

### eduction  
`eduction` lets you capture applying a transducer to a collection. The value returned is an iterable application of the transducer on the collection items which can then be passed to, say, `reduce`.  

{% codeblock %}
(let [xf (comp (map inc)
               (map dec))
      coll (eduction xf (range 10))]
  (reduce + 0 coll))
{% endcodeblock %}  

## Inside a transducible context  
As mentioned before, transducers run in transducible contexts. The ones that come as a part of `clojure.core` would suffice most real-world needs and you'll rarely see yourself writing new ones. Let's look at `transduce`.  

{% codeblock %}
(defn transduce
  ([xform f coll] (transduce xform f (f) coll))
  ([xform f init coll]
     (let [f (xform f)
           ret (if (instance? clojure.lang.IReduceInit coll)
                 (.reduce ^clojure.lang.IReduceInit coll f init)
                 (clojure.core.protocols/coll-reduce coll f init))]
       (f ret))))
{% endcodeblock %}  

`transduce` is just like `reduce`. The 3-arity version expects an initial value to be supplied by calling the 0-arity version of the supplied function. The 4-arity version is slightly more involved. `IReduceInit` is an interface implemented by collections to let them provide an initial value. It lets a collection reduce itself. If not, the call goes to `coll-reduce` which is a faster way to reduce a collection than using first/next recursion.  

## Stateful transducers  
It's possible for transducers to maintain reduction state.   

{% codeblock %}
(defn multiply-xf
  []
  (fn [rf]
    (let [product (volatile! 1)]
      (fn
        ([] (rf))
        ([result] (rf result))
        ([result input]
         (let [new-product (vswap! product * input)]
           (rf result new-product)))))))
{% endcodeblock %}  

Here's a transducer which will multiply all the incoming numbers. We maintain state by using a `Volatile`. Whenever we get a new input we multiply it with the `product` and update the state of `Volatile` using `vswap!`. Let's see this in action:

{% codeblock %}
(into [] (multiply-xf) [1 2 3])
=> [1 2 6]
{% endcodeblock %}

## Early Termination  

The way the above transducer is written, it'll process all the inputs even if one of the inputs is zero. We know that once we encounter a zero, we can safely end the reduction process and return a zero. [`reduced`](https://clojure.github.io/clojure/clojure.core-api.html#clojure.core/reduced) lets you return a reduced value and end the reduction. Let's make a minor change to the above transducer and add in early termination.  

{% codeblock %}
(defn multiply-xf
  []
  (fn [rf]
    (let [product (volatile! 1)]
      (fn
        ([] (rf))
        ([result] (rf result))
        ([result input]
         (let [new-product (vswap! product * input)]
           (if (zero? new-product)
             (reduced result)
             (rf result new-product))))))))
{% endcodeblock %}

In the 2-arity function, we check if the `new-product` is zero. If it is, we know we have a reduced value. We end the reduction by returning the `result` we have so far.  Let's see this in action:

{% codeblock %}
(into [] (multiply-xf) [1 2 3])
=> [1 2 6]
(into [] (multiply-xf) [1 2 0 3])
=> [1 2]
{% endcodeblock %}

## Conclusion  

Transducers can be a very useful tool in your Clojure toolkit that let you process large collections, channels, etc. effectively by letting you make composable transformation pipelines that process one element at a time. They require a little getting used-to but once you're past the learning curve, performance galore!