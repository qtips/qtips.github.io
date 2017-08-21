---
layout: post
title: Effective Java - Generics
tags: effectivejava
enabledisqus: true
---

## Generics (Ch 5)
This is a short summary of Joshua Blochs book [Effective Java](https://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683) chapter 5. I have only included items that I find relevant.

### Don't use raw types (Item 23)
Using unbounded wildcard type is better than using raw types e.g. `Set<?>`. This is because you cannot add anything to such collection other than `null`, while with a raw type you can add anything. So `<?>` are great for readonly collections.

### Prefer Lists to Arrays (Item 25)
Arrays are covariants: If `A` is subtype of `B`, then `A[]` is a subtype of `B[]`. Generics are invariants: `List<A>` is not a subtype of `List<B>`. One might think that arrays are more flexible than generics, but they actually complicate code:

{% highlight java %}
// Fails on runtime, but compiles!
Object[] objectArray = new Long[1];
objectArray[0] = "a string";

// Won't compile
List<Object> o1 = new ArrayList<Long>();
o1.add("a string");
{% endhighlight %}

Because of these differences, arrays and generics do not mix well, and it is illegal to create generic arrays (e.g `new List<E>[]`). Arrays provide runtime safety but not compile time, and vice versa with generics. General advice is to replace arrays with lists when you find you self mixing them. However, if you cannot use lists for example due to performance, there are workarounds, see Item 26.

### Bounded wildcards - extend and super (Item 28)

* Mnemonic for remembering bounded wildcard: **PECS - producer-extends, consumer-super**
  * For example for a stack method `pushAll(srcCollection)` the parameter srcCollection produces elements that are used by the stack, hence the method parameter should use `<? extends T>`.
  * On the other hand, a stack method `popAll(destCollection)` the parameter destCollection consumes elements in the stack, hence the method parameter should use `<? super T>`.
* Do not use wildcard types as return types!
* If a user of a class has to think about wildcard types, there is probably something wrong with the class' API.
* All comparables and comparators are consumers!



