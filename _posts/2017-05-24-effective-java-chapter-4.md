---
layout: post
title: Effective Java - Classes and Interfaces
tags: effectivejava
enabledisqus: true
---

## Classes and Interfaces (Ch 4)
This is a short summary of Joshua Blochs book [Effective Java](https://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683) chapter 4. I have only included items that I find relevant.

### Minimize accessibility to keep your API clean (item 13)
A public class or member is part of the contract to your clients, and you are required to support it forever. Therefore, it is important to start with access level of least accessibility for classes and methods. When defining a public class/member, it is important to think thoroughly about how it will be used by clients.

Aside from the access levels (public, protected, package-private and private), there are some special considerations to look out for regarding accessibility:

*  When using default serialization form of `Serializable` classes, private fields types become part of the public API. If a private field is later removed, the deserializer will not work on old serialized objects.
* `protected` member of a public class are also part of the public API because they are available through extension.
* A non-zero length array is always mutable, so a `public static final` array is modifiable by clients. An alternative is to return a copy with a getter or return a `Collections.unmodifiableList()`

### Object instance field should never be public (item 14)
Instance field should not be declared public and should be accessed through getter, and for mutable classes, setters. A public field can be read and written to any value by clients, and you cannot enforce continuous validation, perform transformations or do other kinds of logic when a value is read or written. However, the worst part is that you cannot change this behaviour later by making the field private and providing accessor methods, or by removing the field. This is because these changes are non-backward compatible, and will break client code.

It is important to note that it may be perfectly fine to expose fields for package-private classes. However, if your package is very large, then refactoring may become a hassle without accessor methods.

If a fields refers to an immutable object, and you are certain that it is here to stay and will _not_ require logic before read and write, then you can provide the field as public final. You can still enforce invariants in the constructor, but you will loose flexibility for future changes.

### Use Immutable Objects (item 15)
Immutable objects are objects have only a single state which is assigned during instance creation as they cannot be modified later. Immutable objects provide several benefits:

* They are **thread-safe**. Since an immutable object cannot be modified, two threads can never interfere with each others read and write operations
* Immutable objects, especially value objects, are often **simpler** than mutable objects because they do not require handling state transition.
* Immutable objects can be shared freely. For example, in a static factory, an existing immutable object can be returned when requested to create an identical object, thus **increasing performance**.
* Defensive copying is used to avoid that clients change the object\'s state from outside the object. There is **no need to defensively copy** immutable objects for securing invariants due to their unchangeable nature.
* There is **no reason to clone or copy** an immutable object because it can be shared freely.
* Immutable objects can **share internal parts**. An example from the book is `BigInteger.negate()` method that return the opposite signed `BigInteger`, where the new `BigInteger` refers to the same internal integer value.
* Immutable objects are great for building other objects, like being used as map keys.

The only real **disadvantage** is that they require a new instance for each state (value) in the state space, which can sometimes seem unnatural and effect performance. You may be required to create multiple intermediate objects which are discarded when you get your desired state. For this reason it is suggested to use **mutable companion class** for which you change the state, but which is only used internally in the. This requires accurately predicting useful states. If the prediction is not possible, you may be forced to make the companion public as an alternative for performance reasons (like `StringBuilder`).

The book mentions these five rules to make a class immutable:

1. No public methods must change instance fields
2. Class must not be extendable, for example by using final classes.
3. All fields must be final. This is especially required when passing reference between threads.
4. All fields must be private, but check out item 13
5. Make defensive copies to prevent changing state. If a immutable object has a getter that returns a mutable type, changes to the returned object should not change the immutable object field.


### Favor composition over inheritance (item 16)
Inheritance drawbacks:

* Inheritance makes subclass dependent on the superclass. If the superclass changes in a later release, it will affect the subclass. This can break the subclass if it is dependent on the superclasses inner workings. The book shows an example of a class extending HashSet to count each time an addition is made to the set, and how this can break if HashSet implements add functionality in a different way (addAll call add for addition).
* Mostly, **issues arise when overriding methods**. Adding new methods is safer, but not bullet proof.
* Inheritance is only appropriate when we have a real is-a relation between two classes.
* Decorator pattern is useful in such context. Often wrapping and exposing a small API instead of inheriting is what you actually need.

There are practically no disadvantages with composition.

### Design and document for inheritance or else prohibit it (item 17)
Classes must explicitly be designed for inheritance:

* Self-use must be documented. I.e. methods calling other overridable methods must be documented, or else they will break
* Construction of object can break if, while constructing, overridden methods are called, and they in turn assume that some fields have been initialized.
  * This also puts limits to `Serializable` and `Cloneable` because they are alternative ways to construct an object.
* Inheritance can require that you document API inner workings which you generally shouldn’t.

Prohibit inheritance of a class if you know the subclasses will break if they extend a class. Alternatively avoid self-us of overridable methods. Avoiding self use can be achieved by refactoring.

### Prefer interfaces over abstract classes (item 18)
* When using interfaces you are not bound to have hierarchical structure
* You can use **mixins** to add small features to your classes, e.g. Comparable
* Skeletal implementation are abstract implementation of non-trivial interfaces (i.e. large interfaces) to help programmers extend an interface. Examples are AbstractList (see the javadoc). Note that item 17 constraints still apply!

### Use interfaces only to define types (item 19)
Apparently, interfaces have been used to gather constants which is bad practice. For constants, add them to classes, use enums or use noninstantiable utility classes.

### Prefer class hierarchies to tagged classes (item 20)
A tagged class is a class which can come in one or more types, but the type is enforced using an instance field. Tagged classes are actually a hierarchical class relation. Use that!

### Favor static member classes over nonstatic (item 22)
There are 4 types of inner classes:

* Static member class
  * “ordinary class” happened to be declared in another class
  * Often used as a public helper class
* Non-static nested class
  * Needs an instance of enclosing class to even be initialized, but construction is usually done by enclosing class.
  * Often used as an Adapter to allow an instance of outer class to be viewed as an instance of some unrelated class (e.g Collection interfaces use this to create Iterators).
* Anonymous class
  * Only one instance needed, without any named type (i.e. Interface type is enough)
* Local class
  * Like anonymous, but need more instances and a type, but only inside a method.

Static member classes do not have access to enclosing class private instance members (however it does have access to static private members), while non-static do. Non-static member classes can therefore hold a reference to enclosing class when it should be garbage collected.

Even if inner classes are declared private, they can be used as return types of public methods from enclosing classes that have access to them. But they cannot be used by the client, except if they are returned as an interface. In the example below, the client can use Inn as a Comparable ass even if it is declared private, but client cannot instantiate it (or even cast it to Inn and use other methods)

{% highlight java %}
public class AClass {
   public Comparable<String> innCreator() {
       return new Inn();
   }
   private static class Inn implements Comparable<String> {
       @Override
       public int compareTo(String o) {
           return 42;
       }
   }
}

{% endhighlight %}



