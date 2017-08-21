---
layout: post
title: Effective Java - Methods Common to All Objects
tags: effectivejava
enabledisqus: true
---

## Methods Common to All Objects (Ch 3)
This is a short summary of Joshua Blochs book [Effective Java](https://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683) chapter 3. I have only included items that I find relevant.

### The general `equals()` contract (item 8)
The equals contract describes the equivalence relation as:

* `x.equals(null) == false`
* **Reflexive** - `x.equals(x) == true`
* **Symmetric** - if `x.equals(y) == true` then `y.equals(x) == true`
* **Transitive** - if `x.equals(y) == true` and `y.equals(z) == true` then `x.equals(z) == true`
* **Consistent** - multiple invocation of equals on the same, unmodified objects, return the same value.

There are some notable properties to be vary of:

* If the above contract is violated, behaviour of other objects (like `List.contains()`) is undefined.
* There is no way to extend an instantiable class with a new value field/component while preserving the equals relation, unless you are willing to forgo the benefits of OO abstractions.
  * Note that in case the base class is abstract, it is fine.
  * An example where this is a problem is Java `Timestamp` class which extends `Date` and violates the symmetry part. If both are intermixed in a Collection, they can create erratic behavior.
  * The solution is prefer composition over inheritance.
* `java.net.URL` relies on IP address of the hosts associated with the URL which require network access, and therefore breaks consistency.
* The book gives recipe for creating an optimal equals:
  1. check if the argument `==this`
  2. use `instance of` to check for correct type
  3. cast to correct type
  4. compare significant fields

### Always override `hashCode()` when you override `equals()` (item 9)
The hashcode is used by hash based structures. The most important part of the hashcode contract states that equal objects must return equal hashcodes. In addition, the hashcode function _should_ return different values for unequal objects for performance. Without a correct hash code implementation, hash based structures will perform poorly and even worse, regard equal objects as unequal. If a constant value is provided as hashCode e.g. `... return 42`, then hash tables degenerate to linked lists and program supposed to run in linear time run on quadratic time.

### Always override `toString()` (item 10)
... because it makes debugging much easier.

### Be cautions of `clone` (item 11)
Implementing `Clonable` makes `Object.clone()` return a field-by-field copy, otherwise it throws `CloneNotSupportedException`. Usually, cloning creates an object, but bypasses the constructor. There are several challenges of implementing `clone`:

* Generally, and especially when extending a class, when overriding `clone`, you should return the object returned by `super.clone()` to get the right type. This is not enforced and it is up to the user to do this, but without it, the clone can break.
* `clone` does not copy mutable object fields, so `super.clone()` will refer to same object fields. Fields must be manually cloned.
  * this essentially means that fields cannot be `final when used with clone, unless the same field value can be shared.
* Since `clone` creates an object without using the constructor, it must ensure that all the invariants are correct after creation.
* `clone` must be called recursively on internal lists/arrays.

The general advise is to avoid using and implementing `Object.clone()` and rather use copy constructors `public Yum(Yum yum)`or factories, except when copying arrays.

### Implementing `Comparable` (item 12)
Comparable deals with order comparison, and is required when using for example `TreeSet`, `TreeMap`, `search` or `sort`.

* Comparable has a similar contract as `equals`, which may lead to erratic behavior may when broken. The contract requires symmetry, reflexivity and transitivity.
* `equals` which is inconsistent with `compareTo` can create duplicates in some collections.
* Float and Double have their own static `compareTo` methods that should ease handling floating point issues.
* Take care when subtracting integers to create return value of `compareTo` because it can create overflow (i.e. outside of `Integer.MAX_VALUE`) and create wrong return value! If `i` is large positive value and `j` is large negative value then `i-j` will overflow and return a negative value.



