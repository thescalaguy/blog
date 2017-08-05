---
title: Scalaz Lens
date: 2017-08-01 16:30:04
tags:
  - scalaz
  - scala
  - functional programming
---
In this post we'll look at [Lens](https://github.com/scalaz/scalaz/blob/5d453fe3e2548f829d26a3b033b82b36c06731fd/core/src/main/scala/scalaz/Lens.scala) which is a pure functional way of getting and setting data. Before we get into lenses, we'll look at why we need lenses by looking at a simple example.  

## Motivating Example  

{% codeblock lang:scala %}
@ case class Address(city: String, zip: Int)
defined class Address
@ case class Person(name: String, address: Address)
defined class Person
{% endcodeblock %}  

Say we have a class `Person` which has the name of the person and their address where the address is represented by `Address`. What we'd like to do is to change the address of the person. Let's go ahead and create a `Person`.

{% codeblock lang:scala %}
@ val p1 = Person("John Doe", Address("Doe Ville", 7))
p1: Person = Person("John Doe", Address("Doe Ville", 7))
{% endcodeblock %}  

Changing the `Address` while maintaining immutability is fairly easy.

{% codeblock lang:scala %}
@ val p2 = p1.copy(
    p1.name,
    Address("Foo City", 9)
  )
p2: Person = Person("John Doe", Address("Foo City", 9))
{% endcodeblock %}  

The problem arises when things begin to nest. Let's create an `Order` class representing an order placed by a `Person`.

{% codeblock lang:scala %}
@ case class Order(person: Person, items: List[String])
defined class Order
@ val o1 = Order(p1, List("shoes", "socks", "toothpaste"))
o1: Order = Order(Person("John Doe", Address("Doe Ville", 7)), List("shoes", "socks", "toothpaste"))
{% endcodeblock %}

Now, the person would like to change the address to which the items are delivered.

{% codeblock lang:scala %}
@ val o2 = o1.copy(
    o1.person.copy(
      o1.person.name,
      Address("Foo City", 9)
    ),
    o1.items
  )
o2: Order = Order(Person("John Doe", Address("Foo City", 9)), List("shoes", "socks", "toothpaste"))
{% endcodeblock %}  

So, the deeper we nest, the uglier it gets. Lenses provide a succinct, functional way to do this.

## Lens

Lenses are a way of focusing on a specific part of a deep data structure. Think of them as fancy getters and setters for deep data structures. I'll begin by demonstrating how we can create and use a lens and then explain the lens laws.  

### Creating a Lens

{% codeblock lang:scala %}
@ import scalaz._
import scalaz._
@ import Scalaz._
import Scalaz._

// creating a lens
@ val addressInPerson = Lens.lensu[Person, Address] (
    (p, address) => p.copy(address = address),
    _.address
  )
addressInPerson: Lens[Person, Address] = scalaz.LensFunctions$$anon$5@4e32e2a2
{% endcodeblock %}  

What we've done is create a lens that accepts a `Person` object and focuses on its `Address` field. `lensu` expects two functions - a setter and a getter. In the first function, the setter, we're making a copy of the `Person` object passed to the lens and updating its address field with the new one. In the second function, the getter, we're simply returning the address field. Lets see this in action by getting and setting values.  

### Getting a Field

{% codeblock lang:scala %}
@ addressInPerson.get(p1)
res12: Address = Address("Doe Ville", 7)
{% endcodeblock %}  

Once you create a lens, you get a `get` method which returns the address field in the `Person` object.

### Setting a Field

{% codeblock lang:scala %}
@ val p3 = addressInPerson.set(p1, Address("Bar Town", 10))
p3: Person = Person("John Doe", Address("Bar Town", 10))
{% endcodeblock %}  

Similarly, there's a `set` method which lets you set fields to specific values.  

### Modifying a Field

{% codeblock lang:scala %}
@ val p4 = addressInPerson.mod({ a => a.copy(city = s"${a.city}, NY") }, p1)
p4: Person = Person("John Doe", Address("Doe Ville, NY", 7))
{% endcodeblock %}

`mod` lets you modify the field. It expects a function that maps `Address` to `Address`. In the example here, we're appending "NY" to the name of the city.

## Lenses are Composable  

The true power of lenses is in composing them. You can compose two lenses together to look deeper into a data structure. For example, we'll create a lens which lets us access the address field of the person in an `Order`. We'll do this by composing two lenses.  

{% codeblock lang:scala %}
// creating the lens
@ val personInOrder = Lens.lensu[cmd5.Order, Person] (
    (o, person) => o.copy(person = person),
    _.person
  )
personInOrder: Lens[cmd5.Order, Person] = scalaz.LensFunctions$$anon$5@33d58abf

// testing the lens
@ personInOrder.get(o1)
res16: Person = Person("John Doe", Address("Doe Ville", 7))
{% endcodeblock %}  

Ignore the `cmd.` prefix to `Order`. That is just an Ammonite REPL quirk to avoid confusing with the `Order` trait from Scalaz. Next, we'll combine the two lenses we have.  

{% codeblock lang:scala %}
@ val addressInOrder = personInOrder >=> addressInPerson
addressInOrder: LensFamily[cmd5.Order, cmd5.Order, Address, Address] = scalaz.LensFamilyFunctions$$anon$4@73679b13
{% endcodeblock %}  

`>=>` is the symbolic alias for `andThen`. The way you read what we've done is: *get the person from the order AND THEN get the address from that person*.  

This allows you to truly keep your code DRY. Now no matter within which data structure `Person` and `Address` are, you can reuse that lens to get and set those fields. It's just a matter of creating another lens or few lenses to access the `Person` from a deep data structure.  

Similarly there's also `compose` which has a symbolic alias `<=<` and works in the other direction. I personally find it easier to use `andThen` / `>=>`.  

## Lens Laws  

**Get-Put:** If you get a value from a data structure and put it back in, the data structure stays unchanged.  
**Put-Get:** If you put a value into a data structure and get it back out, you get the most updated value back.  
**Put-Put:** If you put a value into a data structure and then you put another value in the data structure, it's as if you only put the second value in.  

Lenses that obey all the three laws are called "very well-behaved lenses". You should always ensure that your lenses obey these rules.  

Here's how Scalaz represents these lens laws:

{% codeblock lang:scala %}
trait LensLaw {
  def identity[A >: A2 <: A1, B >: B1 <: B2](a: A)(implicit A: Equal[A]): Boolean = {
    val c = run(a)
    A.equal(c.put(c.pos: B), a)
  }
  def retention[A >: A2 <: A1, B >: B1 <: B2](a: A, b: B)(implicit B: Equal[B]): Boolean =
    B.equal(run(run(a).put(b): A).pos, b)
  def doubleSet[A >: A2 <: A1, B >: B1 <: B2](a: A, b1: B, b2: B)(implicit A: Equal[A]): Boolean = {
    val r = run(a)
    A.equal(run(r.put(b1): A) put b2, r put b2)
  }
}
{% endcodeblock %}

`identity` is get-put law, `retention` is put-get law, and `doubleSet` is put-put law.

## Lenses and State Monads  

Formally, a state monad looks like the following: 

{% codeblock %}
S => (S, A)
{% endcodeblock %}  

Given a state S, it computes the resulting state S by making mutations to the existing state S and produces a resulting A. This is a bit abstract so let's look at a scenario. Say we have a list of people whose addresses we'd like to update to Fancytown with zip code 3. Let's do that using lenses.

### Creating a State

{% codeblock lang:scala %}
@ val state = for {
    p <- addressInPerson %= { add => Address("Fancytown", 3) }
  } yield p
state: IndexedStateT[Id, Person, Person, Address] = scalaz.IndexedStateT$$anon$11@25b51460
{% endcodeblock %}

Here we are creating a state using a `for` comprehension. The `%=` operator accepts a function which maps an `Address` to an `Address`. What we get back is a state monad. Now that we have a state monad, let's use it to update the address.

### Updating the State

Next, let's make person `p1` move to Fancytown. 

{% codeblock lang:scala %}
@ state(p1)
res33: (Person, Address) = (Person("John Doe", Address("Fancytown", 3)), Address("Fancytown", 3))
{% endcodeblock %}

Here we are updating person `p1`'s address. What we get back is a new state `S`, `p1` but with Fancytown address, and the result `A`, the new `Address`. `state(p1)` is the same as `state.apply(p1)`. In short, we're applying that state to a `Person` object.

## Conclusion

This brings us to the end the post on lenses. Lenses are a powerful way to get, set, and modify fields in your data structures. The best part about them is that they are reusable and can be composed to form lenses that focus deeper into the data structure.