---
layout: post
title: Effective Java - Enums and Annotations
tags: effectivejava
enabledisqus: true
---

## Enums and Annotations (Ch 6)
This is a short summary of Joshua Blochs book [Effective Java](https://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683) chapter 6. I have only included items that I find relevant.

### Enum (Item 30)
* Use abstract methods in Enums to give each enum constant specific behaviour.
* Strategy enum pattern: delegate part of behaviour into another enum, and use it as a mandatory argument in enum contructor.

### Do not use ordinal() (item 31)
Never derive a value associated with an enum from its ordinal; store it in an instance field. Because ordinal is the index of the enum constant, its value can change for the same enum if the order is changed.

### Use EnumSet instead of bit fields (item 32)
Bit fields are used to perform set operations such as union en intersection efficiently using bitwise arithmetic.

### Extending enums with interfaces (item 34)
It is rarely necessary to have enums extend other enums. One compelling use case where extension is useful is when implementing opcodes, and adding new opcodes. This can be achieved with enums implementing interfaces, where clients write extensions of enums that implement the interfaces.

### Marker interfaces vs annotations - use marker interface for types (item 37)
`Serializable` is an example of a marker interface. Marker interface are useful to define types, which annotation cannot. Marker interfaces that define types allow you to catch errors at compile time, while annotation are not processed before runtime. Questions to ask when deciding between marker interface and annotation (for class or interface):

  1. Will I need methods that use the class or interface as parameter? Yes -> marker interface
  2. Will the marker only be used on certain interfaces/types? Yes -> marker interface as a sub interface
  3. NO to both -> annotation
