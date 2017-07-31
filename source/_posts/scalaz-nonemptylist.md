---
title: Scalaz NonEmptyList
date: 2017-07-30 21:25:30
tags:
  - scalaz
  - scala
  - functional programming
---
In this post we'll look at [NonEmptyList](https://github.com/scalaz/scalaz/blob/fabab8f699d56279d6f2cc28d02cc2b768e314d7/core/src/main/scala/scalaz/NonEmptyList.scala) which is a singly-linked list guaranteed to be non-empty. Let's begin by seeing how using a NEL (short for `NonEmptyList`) in appropriate places is better than using regular `List`.  

## Contrasting List with NEL

{% codeblock lang:scala %}
@ def email(body: String, recipients: List[String]): Unit = {
    // send email
  }
defined function email
{% endcodeblock %}  

Let's say we're writing an `email` function that sends email to a list of people. A naive assumption would be to always expect `recipients` to have at least one element. However, we _may_ end up with an empty list and that'll lead to painstaking debugging trying to find out why emails weren't sent out.   

So, we'll change our `email` function to this:

{% codeblock lang:scala %}
@ def email(body: String, recipients: List[String]): Either[String, String] = {
    if(recipients.length == 0)
        Left("Empty list of recipients")
    else
        // send email
        Right(s"Email sent to ${recipients.length} recipient(s)")
  }
defined function email
{% endcodeblock %}  

Now although we're handling the case of empty list, we've introduced unnecessary noise with an `if`-`else`. Here's how we can get a guarantee on being non-empty and avoid the conditional:

{% codeblock lang:scala %}
@ import scalaz._
import scalaz._
@ import Scalaz._
import Scalaz._
@ def email(body: String, recipients: NonEmptyList[String]): Unit = {
    // send email
  }
defined function email
{% endcodeblock %}  

Voila! Using a NEL gives us two advantages:
<ul>
	<li> A guarantee on never receiving an empty list </li>
	<li> Expressing our intent in the function signature </li>
</ul>

With that said, let's look at how we can create and use NELs.  

### Creating a NEL

{% codeblock lang:scala %}
@ NonEmptyList(1, 2, 3, 4)
res2: NonEmptyList[Int] = NonEmpty[1,2,3,4]
{% endcodeblock %}

In the example above, we're using the `apply` method to create a NEL. It looks like the following:  

{% codeblock lang:scala %}
def apply[A](h: A, t: A*)
{% endcodeblock %}

The first argument is the head whereas the second argument is varargs which becomes the tail. Internally, the `apply` method calls the `nels` method. We can use that directly, too.

{% codeblock lang:scala %}
@ NonEmptyList nels(1, 2, 3, 4)
res3: NonEmptyList[Int] = NonEmpty[1,2,3,4]
{% endcodeblock %}

You can also create a NEL from a single value by using `wrapNel`.

{% codeblock lang:scala %}
@ 1.wrapNel
res4: NonEmptyList[Int] = NonEmpty[1]
{% endcodeblock %}

You can even create a NEL in the cons list way like you'd do with a `List`

{% codeblock lang:scala %}
// good ol' List
@ 1 :: List(2, 3, 4)
res5: List[Int] = List(1, 2, 3, 4)

// NEL
@ 1 <:: NonEmptyList(2, 3, 4)
res6: NonEmptyList[Int] = NonEmpty[1,2,3,4]
{% endcodeblock %}

You can convert a `List` to a NEL by calling `toNel`. This will yield an `Option` of NEL since the `List` may be empty

{% codeblock lang:scala %}
@ List(1, 2, 3) toNel
res7: Option[NonEmptyList[Int]] = Some(NonEmpty[1,2,3])
{% endcodeblock %}

### Appending NELs

{% codeblock lang:scala %}
@ NonEmptyList(1, 2, 3) append NonEmptyList(4, 5, 6)
res8: NonEmptyList[Int] = NonEmpty[1,2,3,4,5,6]
{% endcodeblock %}  

You can also concatenate two NELs by using `append`.  

### Head and Tail  

{% codeblock lang:scala %}
@ NonEmptyList(1, 2, 3).head
res13: Int = 1
@ NonEmptyList(1, 2, 3).tail
res14: IList[Int] = ICons(2, ICons(3, []))

@ List().head
java.util.NoSuchElementException: head of empty list
  scala.collection.immutable.Nil$.head(List.scala:428)
  scala.collection.immutable.Nil$.head(List.scala:425)
  ammonite.$sess.cmd15$.<init>(cmd15.sc:1)
  ammonite.$sess.cmd15$.<clinit>(cmd15.sc)
{% endcodeblock %}  

`head` and `tail` return and head and tail of the NEL, respectively. Note that `head` is guaranteed to work without exceptions because we'll always have one element.

### Iterating

{% codeblock lang:scala %}
@ NonEmptyList(1, 2, 3) foreach println
1
2
3

@ NonEmptyList(1, 2, 3) map { _ + 1 }
res18: NonEmptyList[Int] = NonEmpty[2,3,4]
{% endcodeblock %}  

Because NEL is also a list, you can perform familiar list operations like `foreach`, `map`, etc.

## Conclusion  

This post was intended to give you an introduction to Scalaz `NonEmptyList` and how it differs from the usual `List`. The guarantee on NEL being non-empty means that you can call `head` without having to worry about exceptions. This makes your code less cluttered and its intent more expressive. 