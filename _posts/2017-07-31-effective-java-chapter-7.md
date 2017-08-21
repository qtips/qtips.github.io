---
layout: post
title: Effective Java - Methods
tags: effectivejava
enabledisqus: true
---

## Methods (Ch 7)
This is a short summary of Joshua Blochs book [Effective Java](https://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683) chapter 7. I have only included items that I find relevant.

### Parameter validation (Item 38)
* Non-public methods should check their parameters using assertions (`assert`) because you as author control the circumstances under which the method are called.
* It is particularly important to check validity of parameters that are stored away for later use.
* Developers should create a habit of checking parameter validity at the start of the method or constructor.

### Make defensive copies when needed (Item 39)
* When programming, assume that clients will do their best to destroy invariants, in other words, create corrupted states and unintentional usage of your objects.
* Create defensive copies of mutable field assignments constructor parameters so that changes on the original object do not change your objects fields.
  * The item mentions security threat where a `Date` object is modified outside of the object.
  * Copies should be made **before** checking the validity so that validity checks are made on copies. This is to protect classes for changes from other threads.
* Do not use clone method to make defensive copies whose type is subclassable by untrusted parties. For example `java.util.Date` is nonfinal.
* Modify field accessors (getters) to return defensive copies of mutable internal fields so that the field does not change its internal values outside of objects control. This again applies to `Date`.
* **To summarize, when possible, use immutable objects as components of your objects**

### Method signature design guidelines (Item 40)
* Don't create to many convenience methods that you think may be used. When in doubt, leave it out.
* Avoid long parameters, max 4-5. Shortning techniques:

  1. break up in smaller methods with fewer parameters instead of providing a do-it-all method. Then client has to mix the methods to get desired purpose. For example, `List` does not contain a method for finding first or last element in a sublist which would require 3 parameters.
  2. combine the parameters into a class/entity
  3. combine 1 and 2 together with `Builder` pattern; when all the parameteres are set in the builder, an "execute" method can perform the calculation.

* For parameter types, favor interfaces over classes (item 52).
* Prefer two-element enums instead of boolean. This increases readability and the enums can be defined in a single line.

### Minimize method overloading (Item 41)
Selection of overloaded methods is done compile-time:
{% highlight java %}
public class CollectionClassifier {
    public static String classify(Set<?> s) { return "Set"; }
    public static String classify(List<?> s) { return "List"; }
    public static String classify(Collection<?> s) { return "Unknown"; }

    public static void main(String[] args) {
        Collection<?>[] collections = {
            new HashSet<String>(),
            new ArrayList<BigInteger>(),
            new HashMap<String,String>().values()};
        for (Collection<?> c: collections) {
            System.out.println(classify(c))
        }
    }
}
{% endhighlight %}

The above prints "Unknown" 3 times because choice of overloading is done compile time, and the compile time type is `Collection<?>` for all 3. The quickest solution is to replace classify with one method that does instance of checking.

* Overriding in subclasses the norm and overloading is the exception
* Overloading methods with same number of parameters should be avoided, instead use different names.

### Return empty arrays or collections, not nulls (Item 43)
There is no reason ever to return null from an array or collection valued method instead of returning an empty array or collection. If performance is an issue, return the same zero length array that always are immutable.