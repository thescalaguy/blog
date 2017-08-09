---
title: Introduction to Category Theory
date: 2017-08-08 22:15:35
tags:
  - category theory
  - scala
  - functional programming
---

We've covered a lot of topics in Scalaz but before moving forward, I'd like to cover functors, monoids, monads, etc. These form the basis of functional programming and are predicated in category theory. This post is intended to be an introduction to category theory.  

## What is Category Theory?

Category theory is a mathematical theory involving the study of categories. A category consists of a group of objects and transformations between them. Think of a category as a simple collection.<sup>[[1]](https://en.wikibooks.org/wiki/Haskell/Category_theory)</sup> 

Formally, a category $C$ consists of the following: 

1. a collection of **objects**
2. a collection of **arrows** (called **morphisms**)
3. operations assigning each arrow $f$ an object $dom \space f$, its **domain**, and an object $cod \space f$, its **codomain**. We write this as $f: A \rightarrow B$
4. a composition operator assigning each pair of arrows $f$ and $g$, with $cod \space f = dom \space g$ a composite arrow $g \circ f: dom \space f \rightarrow cod \space g$, satisfying the _associative law_:   
 for any arrows $f: A \rightarrow B$, $g: B \rightarrow C$, and $h: C \rightarrow D$ (with $A$, $B$, $C$, and $D$ not necessarily distinct),
 $h \circ (g \circ f) = (h \circ g) \circ f$
5. for each object $A$, an **identity arrow** $id_A: A \rightarrow A$ satisfying the _identity law_:
 for any arrow $f: A \rightarrow B$,
 $id_B \circ f = f$ and $f \circ id_A = f$

The formal definition above is taken verbatim from [Basic Category Theory for Computer Scientists](https://mitpress.mit.edu/books/basic-category-theory-computer-scientists).

<center>
![Simple Category](https://upload.wikimedia.org/wikibooks/en/d/d3/Simple-cat.png)
</center>  

Let's relate the diagram above<sup>[[2]](https://en.wikibooks.org/wiki/Haskell/Category_theory)</sup> to the formal definition that we have. This simple category $C$ has three **objects** $A$, $B$, and $C$. There's three **identity arrows** $id_A$, $id_B$, and $id_C$. These identity arrows satisfy the **identity law**. For example, $id_A \circ g = g$. Intuitively, if you were "standing" on $A$ and you first "walked along" the $id_A$ arrow and then "walked along" the $g$ arrow to reach $B$, it's as good as just "walking along" $g$. 

## A More Concrete Example

Let's consider a category $S$ whose objects are sets. We'll translate this into code and hold it to the laws stated above. 

1. $S$ is a collection of sets i.e. each object is a set.
2. an arrow $f: A \rightarrow B$ is a morphism from set $A$ to set $B$
3. for each function $f$, we have $dom \space f = A$, and $cod \space f = B$
4. the composition of a function $f: A \rightarrow B$ with $g: B \rightarrow C$ is a function from $A$ to $C$ mapping each element $a \in A$ to $g(f(a)) \in C$
5. for each set $A$, the identity function $id_A$ is a function with domain and codomain as $A$.

### Code

Let's begin by creating our first object of category $S$ - a set $A$.

{% codeblock lang:scala %}
@ val A = Set("apples", "oranges")
A: Set[String] = Set("apples", "oranges")
{% endcodeblock %} 

Next, let's define a function $f$ with morphs $A$ to $B$.

{% codeblock lang:scala %}
@ def f(a: Set[String]): Set[String] = a map { _.reverse }
defined function f
{% endcodeblock %}

Next, let's morph $A$ to $B$ by applying the function $f$

{% codeblock lang:scala %}
@ val B = f(A)
B: Set[String] = Set("selppa", "segnaro")
{% endcodeblock %}

The domain of $f$ is the set $A$ where as codomain is the set of reversed strings, $B$.

Next, let's define a function $g$ 

{% codeblock lang:scala %}
@ def g(b: Set[String]): Set[Int] = b map { _.length }
defined function g
{% endcodeblock %}

Now let's compose $f$ and $g$

{% codeblock lang:scala %}
@ val C = g(f(A))
C: Set[Int] = Set(6, 7)
{% endcodeblock %}

And finally, let's create an identity function

{% codeblock lang:scala %}
@ def idA(a: Set[String]): Set[String] = a map identity
defined function idA
{% endcodeblock %}

Let's see this in action

{% codeblock lang:scala %}
@ idA(A)
res7: Set[String] = Set("apples", "oranges")
{% endcodeblock %}

This is how we translate a category to code. In the coming posts we'll cover more category theory. 