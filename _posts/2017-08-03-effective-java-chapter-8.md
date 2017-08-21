---
layout: post
title: Effective Java - General Programming
tags: effectivejava
enabledisqus: true
---

## General Programming (Ch 8)
This is a short summary of Joshua Blochs book [Effective Java](https://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683) chapter 8. I have only included items that I find relevant.

### Avoid float and double if exact answers are required (Item 48)
In binary floating point arithmetic floats and doubles cannot represent 0.1 or any (10^-x) exactly.
{% highlight java %}
// Broken - uses floating point for monetary calculation!
public static void main(String[] args) {
   double funds = 1.00;
   int itemsBought = 0;
   for (double price = .10; funds >= price; price += .10) {
      funds -= price;
      itemsBought++;
   }
   System.out.println(itemsBought + " items bought.");
   System.out.println("Change: $" + funds); // $0.3999999999999999
}

{% endhighlight %}

To remedy this, use `BigDecimal` (and its methods), `int` or `long` for monetary calculations.
* `BigDecimal` provides eight rounding modes whenever an operation that entails rounding is performed, however it can be slow
* For performance, use int or long and keep track of the decimal point yourself.

### Primitives and boxed primitives types (Item 49)
Java does alot under the hood on boxed types, and there are some pitfalls:
* Applying the == operator to boxed primitives is almost always wrong! Because it compares the object id. However, < and > operator work as expected.
* In nearly every case, when you mix primitives and boxed primitives, the boxed primitives is auto unboxed. When unboxing, a NullPointerException can be thrown.

### Beware of performance of string concatenation (Item 51)
In newer versions of java, string concatenation using plus operator is replaced by a StringBuffer by the compiler, so less care has to be taken. However, this is not always possible. The following example is not optimized by the compiler:
{% highlight java %}
/ Inappropriate use of string concatenation - Performs horribly!
public String statement() {
   String result = "";
   for (int i = 0; i < numItems(); i++)
      result += lineForItem(i);  // String concatenation
   return result;
}
{% endhighlight %}

When the appended String is initialized outside a loop and appended to inside the loop, the compiler cannot determine how much string concatenation is being done.

### Refer to objects by their interface (Item 52)
* Generally, objects should refer to interfaces if it exists.
* However, especially for value classes (like String or Integer), it may not make sense to refer to interface.

### Reflection and minimizing usage through interface (Item 53)
Reflection was designed for application creating application builder tools, so it should not be used by normal applications. Examples of applications for which reflection can be appropriate are class browsers, object inspectors, code analysis tools and interpretive embedded systems.

Drawbacks of using reflection:

* You loose all benefits of compile time checking
* Reflection code is clumsy and verbose
* Performance suffers. On a regular machine the difference for a method call can be as small as 2x to 50x.

Many programs that require a class that is unavailable at compile time can use interfaces to refer to the class, but use reflection to instantiate the object. This way the method calls do not need to be called reflectively.

### Use native methods judiciously (Item 54)
It is rarely advisable to use native methods for improved performance because JVM does a good job by itself. Disadvantage of using native methods:

* Native methods are not immune to memory corruption errors
* Native methods are not necessarily portable because they are platform dependant.
* There is a fixed cost of going in and out of native methods, so they can actually decrease performance
* Native methods require "glue code" that can be verbose and difficult to read.

### Be careful when optimizing code (Item 55)
Golden rule: Strive to write good programs rather than fast one. However, a bad upfront design can really hurt performance and be difficult to change later. Therefore it is essential to consider performance impact when designing. Especially consider parts involved in interaction between modules and between the outside world:

* API design
 * public mutable type may require a lot of needless defensive copying (item 39)
 * inheritance ties the class forever to its superclass (item 16)
 * using implementation types rather than interface type ties you to a specific implementation (item 52), making it difficult to replace with a faster implementation later.
* wire level protocols
* persistent data formats

These are difficult/impossible to change after release. The JDK suffers decisions for example in the design of `Dimension.getSize()`

Optimization tips:

* always measure execution time before and after optimizations using a profiler.
* JVM performance can vary from implementation to implementation, release to release and processor to processor.