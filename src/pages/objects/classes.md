---
layout: page
title: Classes
---

A class is a template for creating objects that have similar methods and fields. Importantly, in Scala a class is also a type. This helps us overcome the problem we had in the *Greetings, Humans* exercise in the last section.

## Defining a Class

Here is a definition for a simple `Person` class:

~~~ scala
scala> class Person {
     |   val firstName = "Noel"
     |   val lastName = "Welsh"
     |   def name = firstName + " " + lastName
     | }
defined class Person
~~~

We can create a new `Person` object using the `new` operator as follows and access its methods and fields in the usual way:

~~~ scala
scala> val noel = new Person
noel: Person = Person@3235186a

scala> noel.firstName
res24: String = Noel
~~~

Notice the type of the object is `Person`. Each call to `new` creates a distinct object of the same type:

~~~ scala
scala> noel // noel the object that prints '@3235186a'
res25: Person = Person@3235186a

scala> val newNoel = new Person // each new object prints a new number
newNoel: Person = Person@2792b987

scala> val anotherNewNoel = new Person
anotherNewNoel: Person = Person@63ee4826
~~~

This means we can write a method that takes aany `Person` as a parameter:

~~~ scala
scala> object alien {
     |   def greet(p: Person) =
     |     "Greetings, " + p.firstName + " " + p.lastName
     | }
defined module alien

scala> alien.greet(noel)
res5: String = Greetings, Noel Welsh

scala> alien.greet(newNoel)
res6: String = Greetings, Noel Welsh
~~~

<div class="alert alert-info">
**Java tip:** Scala classes are all subclasses of `java.lang.Object` and are, for the most part, usable from Java as well as Scala. The default printing behaviour of `Person` comes from the `toString` method defined in `java.lang.Object`.
</div>

## Constructors

As it stands our `Person` class is rather useless: we can create as many new objects as we want but they all have the same `firstName` and `lastName`. What if we want to give each person a different name?

The solution is to introduce a **constructor**, which allows us to pass parameters to new objects as we create them:

~~~ scala
scala> class Person(first: String, last: String) {
     |   val firstName = first
     |   val lastName = last
     |   def name = firstName + " " + lastName
     | }
defined class Person

scala> val dave = new Person("Dave", "Gurnell")
dave: Person = Person@3ed12df7

scala> dave.name
res29: String = Dave Gurnell
~~~

The constructor parameters `first` and `last` are local variables that can only be used within the body of the class. We must declare a field or method using `val` or `def` to access data from outside the object.

Constructor arguments and fields are often redundant. Fortunately, Scala provides us a useful short-hand way of declaring both in one go. We can prefix the constructor parameters with the `val` keyword to have Scala define fields for them automatically:

~~~ scala
scala> class Person(val firstName: String, val lastName: String) {
     |   def name = firstName + " " + lastName
     | }
defined class Person

scala> new Person("Dave", "Gurnell").name
res29: String = Dave Gurnell
~~~

<div class="alert alert-info">
**Tip:** `val` fields are *immutable* -- they are initialized once after which we cannot change their values. Scala also provides the `var` keyword for defining *mutable* fields.

Scala programmers tend to prefer to write immutability and side-effect-free code so we can reason about it using the substitution model. In this course we will concentrate almost exclusively on immutable `val` fields.
</div>

## Default and Keyword Parameters

All Scala methods and constructors support *keyword parameters* and *default parameter values*.

Whenever we call a method or constructor, we can **use parameter names as keywords** to specify the parameters in an arbitrary order:

~~~ scala
scala> new Person(lastName = "Last", firstName="First")
res10: Person = Person(First,Last)
~~~

This comes in doubly useful when used in combination with **default parameter values**, defined like this:

~~~ scala
scala> def greet(firstName: String = "Some", lastName: String = "Guy") =
     |   "Greetings, " + firstName + " " + lastName + "!"
greet: (firstName: String, lastName: String)String
~~~

If a parameter has a default value we can omit it in your call:

~~~ scala
scala> greet("Awesome")
res12: String = Greetings, Awesome Guy
~~~

Combining keywords with default parameter values let us skip earlier parameters and just provide values for later ones:

~~~ scala
scala> greet(lastName = "Dave")
res11: String = Greetings, Some Dave!
~~~

<div class="alert alert-info">
**Design tip:** *Keyword parameters are robust to changes in the number and order of parameters.* For example, if we add a `title` parameter to the `greet` method, the meaning of keywordless method calls changes but keyworded calls remain the same:

~~~ scala
scala> def greet(title: String = "Mr", firstName: String = "Some", lastName: String = "Guy") =
     |   "Greetings, " + title + " " + firstName + " " + lastName + "!"
greet: (firstName: String, lastName: String)String

scala> greet("Awesome") // this is now incorrect
res12: String = Greetings, Awesome Some Guy

scala> greet(firstName = "Awesome") // this is still correct
res12: String = Greetings, Mr Awesome Guy
~~~

This is particularly useful when creating methods and constructors with large number of parameters.
</div>

## Take Home Points

In this section we learned how to define **classes**, which are a kind of **type**.

Classes let us *abstract across objects* that have similar properties.

The properties of the objects of a class take the form of **fields** and **methods**. Fields are pre-computed values stored within the object and methods are computations we can call.

Finally, we learned the main syntactic rules about defining methods:

 - **parameters require types** and can optionally have **default values**;
 - **return types may be omitted** in many cases;
 - all methods can be called using **keyword parameters**.

## Exercises

We now have enough machinery to have some fun playing with classes.

### A Simple Counter

Implement a `Counter` class. The constructor should take an `Int`. The methods `inc` and `dec` should increment and decrement the counter respectively returning a new `Counter`. Here's an example of the usage:

~~~ scala
scala> new Counter(10).inc.dec.inc.inc.count
res42: Int = 12
~~~

<div class="solution">
~~~ scala
class Counter(val count: Int) {
  def dec = new Counter(count - 1)
  def inc = new Counter(count + 1)
}
~~~

Aside from practicing with classes and objects, this exercise has a second goal -- to think about why `inc` and `dec` return a new `Counter`, rather than updating the same counter directly.

Because `val` fields are immutable, we need to come up with some other way of propagating the new value of `count`. Methods that return new `Counter` objects give us a way of returning new state without the side-effects of assignment. They also permit *method chaining*, allowing us to write whole sequences of updates in a single expression

<div class="alert alert-info">
**Performance tip:** The use-case `new Counter(10).inc.dec.inc.inc.count` actually creates 5 instances of `Counter` before returning its final `Int` value. You may be concerned about the extra memory and CPU overhead for such a simple calculation, but don't be. Modern execution environments like the JVM render the extra overhead of this style of programming negligable in all but the most performance critical code.
</div>
</div>

### Counting Faster

Augment the `Counter` from the previous exercise to allow the user can optionally pass an `Int` parameter to `inc` and `dec`. If the parameter is omitted it should default to `1`.

<div class="solution">
The simplest solution is this:

~~~ scala
class Counter(val count: Int) {
  def dec(amount: Int = 1) = new Counter(count - amount)
  def inc(amount: Int = 1) = new Counter(count + amount)
}
~~~

However, this adds parentheses to `inc` and `dec`. If we omit the parameter we now have to provide an empty pair of parentheses:

~~~ scala
scala> new Counter(10).inc
<console>:9: error: missing arguments for method inc in class Counter;
follow this method with `_' if you want to treat it as a partially applied function
              new Counter(10).inc
                              ^
~~~

We can work around this using *method overloading* to recreate our original parenthesis-free methods. Note that overloading methods requires us to specify the return types:

~~~ scala
scala> class Counter(val count: Int) {
     |   def dec: Counter = dec()
     |   def inc: Counter = inc()
     |   def dec(amount: Int = 1): Counter = new Counter(count - amount)
     |   def inc(amount: Int = 1): Counter = new Counter(count + amount)
     | }
defined class Counter

scala> new Counter(10).inc.inc(10).count
res15: Int = 21
~~~
</div>

### Additional Counting

Here is a simple class called `Adder`.

~~~ scala
class Adder(amount: Int) {
  def add(in: Int) = in + amount
}
~~~

Extend `Counter` to add a method called `adjust`. This method should accept an `Adder` and return a new `Counter` with the result of applying the `Adder` to the `count`.

<div class="solution">
~~~ scala
class Counter(val count: Int) {
  def dec = new Counter(count - 1)
  def inc = new Counter(count + 1)
  def adjust(adder: Adder) =
    new Counter(adder.add(count))
}
~~~

This is an interesting pattern that will become more powerful as we learn more features of Scala. We are using `Adders` to capture computations and pass them to `Counter`. Remember from our earlier discussion that *methods are not expressions* -- they cannot be stored in fields or passed around as data. Our `Adders`, however, give us the benefits of being an object as well as a unit of functionality.

Using objects as computations is a common paradigm in object oriented programming languages. Consider, for example, the classic `ActionListener` from Java's Swing:

~~~ java
public class MyActionListener implements ActionListener {
  public void actionPerformed(ActionEvent evt) {
    // Do some computation
  }
}
~~~

The disadvantage of objects like `Adders` and `ActionListeners` is that they are limited to use in one particular circumstance. Scala includes a much more general concept called *functions* that allow us to represent any kind of computation as an object. We will be introduced to functions gradually throughout this chapter.
</div>