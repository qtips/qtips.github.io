---
layout: post
title: Effective Java - Concurrency
tags: effectivejava
enabledisqus: true
---

## Concurrency (Ch 10)
This is a short summary of Joshua Blochs book [Effective Java](https://www.amazon.com/Effective-Java-2nd-Joshua-Bloch/dp/0321356683) chapter 10. I have only included items that I find relevant.

### Synchronize access to shared mutable data (Item 66)
Reading or writing a variable other than `long` or `double` is atomic, so why use `synchronized`? Synchronized is required for reliable communication between threads as well as for mutual exclusion.

{% highlight java %}
// Broken! - How long would you expect this program to run?
public class StopThread {
    private static boolean stopRequested;
    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(new Runnable() {
            public void run() {
                int i = 0;
                while (!stopRequested)
                    i++;
            }
        });
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        stopRequested = true;
    }
}
{% endhighlight %}

Here, one thread is stopping the other using a boolean flag. This program will run forever, because jvm can optimize (hoisting)

{% highlight java %}
while (!stopRequested)
    i++;
{% endhighlight %}
to
{% highlight java %}
if (!stopRequested)
    while (true)
        i++;
{% endhighlight %}

Fix:

{% highlight java %}
// Properly synchronized cooperative thread termination
// Both read and write operation has to be synchronized for synchronization to have effect
public class StopThread {
    private static boolean stopRequested;
    private static synchronized void requestStop() {
        stopRequested = true;
    }
    private static synchronized boolean stopRequested() {
        return stopRequested;
    }
    public static void main(String[] args)
            throws InterruptedException {
        Thread backgroundThread = new Thread(new Runnable() {
            public void run() {
                int i = 0;
                while (!stopRequested())
                    i++;
            }
        });
        backgroundThread.start();
        TimeUnit.SECONDS.sleep(1);
        requestStop();
    }
}
{% endhighlight %}

So, in this case, the use of `synchronized` is to solely to communicate between threads, and not mutual exclusion, because the actions in the `synchronized` methods would be atomic even without the keyword.

In this case however, **a better solution** is to declare `stopRequested` as `volatile` which is less verbose and performs better. `volatile` performs no mutual exclusion but guarantees that any thread that reads the field will see the most recently written value.

`volatile` won't work in the following case:
{% highlight java %}
private static volatile int nextSerialNumber = 0;
public static int generateSerialNumber() {
    return nextSerialNumber++
}
{% endhighlight %}
Here, the `nextSerialNumber` is first read and then incremented. Consider if a thread is generating a new serial number. If a second thread reads the value at the moment the first thread is reading before incrementing, the second thread gets an old value. There are two solutions: remove `volatile` and use `synchronize` or remove `volatile` and use `AtomicLong`. The best solution however, is to avoid sharing mutable data!

### Avoid excessive synchronization (item 67)
Synchronization can cause reduced performance, deadlock or even nondeterministic behaviour.

* Inside a synchronized region, do not invoke a method that is designed to be overridden, or one provided by a client in the form of a function object. If that method indirectly (e.g. by calling another method in another thread) requires a lock to an object already locked by the synchronized block, a deadlock occurs.
* Solution to the above problem is to move the alien invocation outside of synchronized block. A better solution can be to use `CopyOnWriteArrayList` where all write operation perform a copy of the underlying array. This method can avoid synchronized blocks entirely (see item for example).
* Do as little as possible inside synchronized regions.
* Performance cost of synchronization: Lost opportunities of CPU parallelism, and limiting JVM optimizations.
* When in doubt do not synchronize, but document thread-unsafety and let the client do synchronization. However, if you know that your objects will be used in multithreaded environments, provide threadsafety (or threadsafe versions) because internal locking is faster than externally locking the entire object.

### Prefer executors and tasks to threads (item 68)
Executors and tasks can be used instead of threads to execute parallel execution.

* Use `Executors.newCachedThreadPool` for small programs or light loaded servers because it demands no configurations and has a good default. It is not good for production because it does not queue threads and floods the CPU.
* Use `Executors.newFixedThreadPool` for production.
* Use `ScheduledThreadPoolExecutor` for timed executions.

### Prefer concurrency utilities to wait and notify (item 69)
Higher level concurrency utilities: The executor framework (item 68), concurrent collections and Synchronizers.

**Concurrent Collection:**

* Concurrent collections manage their own synchronization internally.
* Some concurrent collections are extended with state-dependent modify operations like `ConcurrentMap.putIfAbsent`
* use `ConcurrentHashMap` in preference to `Collections.synchronizedMap` or `HashTable`.

**Synchronizers** are objects that enable threads to wait for one another for coordination. Example: CountDownLatch and Semaphore.

* For interval timing, always use `System.nanoTime` in preference to `System.currentTimeMillis` because it is more accurate, precise and not affected by adjustments to the system's real-time clock.

_Always use the utilities mentioned above instead of wait and notify as there is seldom any reason to use them in new code._

### Use lazy initialization judiciously (item 71)
If you must initialize a field lazily in order to achieve performance goals, or to break a harmful initialization circularity, then use appropriate lazy initialization techniques. For instance fields, it is the double-check idiom; for static fields, the lazy initialization holder class idiom. For instance fields that can tolerate repeated initialization, you may also consider the single-check idiom.

### Do not depend on the thread scheduler (item 72)
A thread scheduler determines which thread gets to execute next. Do not make assumptions and depend on thread scheduler policy when programming because it is likely to be non-portable (OS).
