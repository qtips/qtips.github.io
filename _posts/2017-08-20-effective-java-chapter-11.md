---
layout: post
title: Effective Java - Serialization
tags: effectivejava
enabledisqus: true
---

## Serialization (Ch 11)
This is a short summary of Joshua Blochs book [Effective Java](https://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683) chapter 11. I have only included items that I find relevant.

### Be careful when implementing Serializable (item 74)
Costs of implementing `Serializable`:

* It decreases the flexibility to change a class's implementation once it has been released, because the serialized form because a part of the exported API. You must then support a released serialized form forever if not properly designed.
  * The default serialization makes the private and package-protected instance fields part of the exported API because it is inside the serialized form.
  * You should provide an explicit serial version UID instead of using the system generated one because it breaks compatibility of older class version, because a new UID is generated when altering a class.
* It increases the likelihood of bugs and security holes as deserialization is a hidden constructor and must therefore ensure that invariants are intact. More in item 76
* It increases the testing burden associated with releasing a new version of a class because you have to test that the new implementation will still allow deserializing from the old version, and vice versa.

* Classes designed for inheritance should rarely implement `Serializable`, and interfaces should rarely extend it, see the item for more details.
* Inner classes (item 22) should not implement `Serializable`.

### Be careful when accepting default serialized form (item 75)
* Always consider whether the default serialized form is appropriate. It is likely to be appropriate if an object's physical representation is identical to its logical content, for example all the instance fields of an object are used by clients to represent different object states.
* Using default serialization form on object that differ physically and logically have the following disadvantages:
 * It permanently ties the exported API to the current internal representation (which may only be implementation details not for clients).
 * It can consume excessive space because some parts of the object instance fields may only be for temporary usage.
 * It can consume excessive time because of large object graph traverses.
 * It can cause stack overflows for large object graphs.
* You must often provide `readObject()` method to ensure invariants and security even if the default serialization form is appropriate.

**Transient**
Transient field are ignored by the default serialization mechanism.

* If all instance fields are `transient`, it is technically permissible to dispense with invoking `defaultReadObject` and `defaultWriteObject` but you still loose flexibility for future releases.
* Before deciding to make a field non-transient, convince yourself that the field is part of the logical state of the object. If custom serialized form is used, most or all fields should be `transient`.

### Write `readObject` methods defensively (item 76)
* When writing these methods, assume that you are writing a constructor.
* Do not assume that a bytestream is an actual serialized instance.
* Do not invoke overridable methods inside these methods

Check the item summary for a guidelines on writing such method.

### Use serialization proxies to avoid serialization problems (item 78)
The serialization proxy pattern uses private static nested proxy class for serializing and deserializing, instead of using the original class. It must consist of the logical fields of the original class. The serialization is done on the proxy, while upon deserialization the proxy creates an instance of the original class. It is similar to defensive copying.

* The pattern greatly reduces risk of bugs and security problems.
* The drawback is performance - in the item 14% more times is used to serialize and deserialize.
* This pattern should always be used what you find yourself writing `readObject` or `writeObject` methods on a class that is not extendable by its clients.





