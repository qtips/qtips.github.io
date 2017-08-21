---
layout: post
title: Effective Java - Exceptions
tags: effectivejava
enabledisqus: true
---

## Exceptions (Ch 9)
This is a short summary of Joshua Blochs book [Effective Java](https://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683) chapter 9. I have only included items that I find relevant.

### Throw exceptions appropriate to the abstraction (Item 61)
Higher layers should translate lower-level exceptions to their domain. Lower level exception may not make any sense to the client. Further, lower-level exceptions may come from a library that may change over time. Not translating the exceptions may therefore break or pollute the API. However, exception translation should not be overused, as the best way to deal with exceptions is to not let them happen!

### Strive for failure atomicity (Item 64)
Generally, a failed method invocation should leave the object in the state that it was prior to the invocation. Ways to achieve this:

* With immutable objects, failure atomicity is free because failure will never leave an existing object in a corrupted state.
* Do validations and throw exception before altering object state.
* Create temporary copy and use it to replace when operation is complete.

### Don't ignore exceptions (Item 65)
Never leave a `catch` clause empty, at least write a comment.