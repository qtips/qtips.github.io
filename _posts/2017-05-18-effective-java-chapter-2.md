---
layout: post
title: Effective Java - Creating and Destroying Objects
tags: effectivejava
enabledisqus: true
---

## Creating and Destroying Objects (Ch 2)
This is a short summary of Joshua Blochs book [Effective Java](https://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683) chapter 2. I have only included items that I find relevant.

### Static Factories (item 1)
Some advantages of static factories vs constructors:

* Factory methods have names which add a description to the constructor
* They can return pre-constructed object instead of always returning new.
* They can return any sub-type of the declared return type, even non-public classes.

The main disadvantage of using _only_ static factories (e.g. with a private constructor) is that the class cannot be subclassed.

### Builders instead of constructors with many parameters (item 2)
Calling a constructor with many parameters can be cumbersome as it requires looking at the method declaration to understand what the parameters represent. This makes both reading and calling such a constructor difficult.

*One alternative* to this is to construct an object using a parameterless constructor, and then use setters to set the required fields (the JavaBean pattern). The drawback with this approach is that the object can be in an inconsistent state while the invariant are being set. In addition, since you are providing setters, the objects are not mutable making thread safety difficult.

Builders are a second alternative with the best from both worlds. A builder first "collects" the parameters in a readable and compact way, and then instantiate the object by first validating that the invariants are ok.

Since builders can be overkill for small classes, the book recommends using it for classes with more than four parameters. Note that builders are an alternative for both constructors and static factories.

### Some gotchas with Singelton (item 3)
* Singeltons make it difficult to **test** its clients if the singelton does not implement an interface because you cannot mock out the singelton.
* If singeltons are made **serializable**, they are not singeltons anymore unless special care is taken when deserializing.
  * The best way to implement a singleton is using a **single-element enum** type, which avoids the problem with serialization.


### Avoid creating unnecessary objects (item 5)
Care should be taken when creating objects. Reuse expensive objects, but not on the expense of defensive copying (for immutability - item 39).
Primitives should be **preferred** over boxed primitives, and take care when autoboxing is performed to remove unnecessary object creation.

### Eliminate obsolete object references (item 6)
The books shows a stack example where a poped items are not nulled from the internal array, making them obsolete references that cannot be garbage collected.
Commons sources of memory leaks and obolete references:

* Whenever a class **manages its own memory**, the programmer should be alert for memory leaks.
* Forgotten **caches** entries. Consider using `WeakHashMap` for caches which only holds items if there is an outside reference to the entry.
* **Listeners** and **callbacks** due to lack of de-registration. Also here a `WeakHashMap` can be used.