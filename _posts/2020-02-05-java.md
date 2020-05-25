---
title: Concurrency&#58; Thread Safety In Java
author: Hamza Belmellouki
categories: [Java]
tags: [core-java, concurrency]
comments: true
---  

What is Thread safety? To save you the time from looking up to Wikipedia here is the definition:
> **Thread safety** is a computer programming concept applicable to multi-threaded code. Thread-safe code only manipulates shared data structures in a manner that ensures that all threads behave properly and fulfill their design specifications without unintended interaction.

What does this definition really mean? How can we correctly write thread-safe code? What are some of the constructs that Java offers to write such code?

## Accessing shared data

Access to mutable shared data requires synchronization which can be implemented using one of these approaches:

* Java’s built-in synchronized keyword

* *volatile* fields to guarantee that any thread that reads a field will see the most recently written value

* Explicit locks

* Atomic variables

**Producing Thread-safe code is all about managing access to shared mutable states**. When mutable states are published or shared between threads, they need to be synchronized to avoid bugs like *race conditions* and *memory consistency errors*.

## Race condition

Java Concurrency In Practice defines race condition as:
> A race condition occurs when the correctness of a computation depends on the relative timing or interleaving of multiple threads by the runtime; in other words, when getting the right answer relies on lucky timing.

To understand what the definition means, let us look at an example:
```java
public class CheckThenActRace { // Risky
    private Connection connection = null;

    public Connection getConnection() {
        if (connection == null)
            connection = new Connection();
        return connection;
    }
}
```

`CheckThenActRace` has race conditions that can compromise its correctness. Say that threads A and B execute `getConnection` simultaneously. A sees that `connection` is `null`, and instantiates a new `Connection`. B also checks if `connection` is `null`. Whether `connection` is `null` at this point depends unpredictably on CPU scheduler timing. If `connection` is `null` when B examines it, the two callers to `getConnection` may receive two different results, even though `getConnection` is always supposed to return the same instance.

To resolve such an issue, the operation check-then-act needs to be *atomic*; meaning it should happen all at once. We can achieve atomicity by making `getConnection` synchronized:
```java
public synchronized Connection getConnection() {
    if (connection == null)
        connection = new Connection();
    return connection;
}
```

Now when Thread A calls `getConnection`; A acquires the intrinsic lock of the object and locks it until `getConnection` is completely executed, then A releases the lock for Thread B to acquire it.

## Memory consistency errors

When reads and writes occur in different threads, issues like memory consistency errors could emerge. In Java, there is no guarantee that the reading thread will see a value written by another thread on a timely basis, or even at all.

To enable visibility of memory writes across threads, you need to rely on synchronization. For example:
```java
class MemoryErrorDemo { // don't do this
    private static boolean flag;
    private static int number;

    private static class ReaderThread extends Thread {
        public void run() {
            while (!flag) {
                // do nothing
            }
            System.out.println(number);
        }
    }

    public static void main(String[] args) {
        new ReaderThread().start();
        number = 5;
        flag = true;
    }
}
```

`MemoryErrorDemo` could loop forever because the value of `flag` might never become visible to the reader thread. Even more weirdly, `MemoryErrorDemo` could print zero instead of 5 because the write to `flag` might be made visible to the reader thread before the write to `number`, a phenomenon known as *reordering*.

We could use the built-in intrinsic lock or the volatile keyword, a weaker form of synchronization, to ensure that writes to a variable are propagated predictably to other threads. To make the program works as expected, we have to add the volatile keyword to the second and third line:
```java
private volatile static boolean flag;
private volatile static int number;
```

## Atomicity

Atomicity is when a set of actions all execute as a single operation, in an indivisible manner. Thinking that a set of actions in a multi-threaded program will be executed in sequential order might lead to behavior incorrectness. That’s due to **thread interference**, which means threads may overlap if they execute several steps on the same data. For example:
```java
class AtomicityDemo {

    private static long counter = 0;

    private static class CounterThread extends Thread {
        public void run() {
            for (int i = 0; i < 1000; i++) {
                increment();
            }
        }
    }

    private static void increment() {
        ++counter;
    }

    public static void main(String[] args) throws InterruptedException {
        CounterThread counterThread = new CounterThread();
        counterThread.start();
        for (int i = 0; i < 1000; i++) {
            increment();
        }
        counterThread.join();
        System.out.println("Final number should be: 2000, but it's :" + counter);
    }
}
```

`AtomicityDemo` simulates two threads writing to a shared variable. You may get the correct result if you’re lucky but if you run the program multiple times you’ll get different incorrect values. That’s because the operation `++counter` isn’t a one-time operation, instead, it is composed of three steps so thread interference happens between these action steps.

To solve this problem we’ll convert the multi-step increment operation into an atomic operation. Fortunately, the Java core API provides Thread-safe objects to perform the operation. In this case, we’ll leverage `AtomicLong` wich is in `java.util.concurrent.atomic` package. Java Concurrency In Practice states:
> Where practical, use existing thread-safe objects, like AtomicLong, to manage your class’s state. It is simpler to reason about the possible states and state transitions for existing thread-safe objects than it is for arbitrary state variables, and this makes it easier to maintain and verify thread safety.

The resulting code would look like:
```java
import java.util.concurrent.atomic.AtomicLong;

class AtomicityDemo {

    private static AtomicLong counter = new AtomicLong(0);

    private static class CounterThread extends Thread {
        public void run() {
            for (int i = 0; i < 1000; i++) {
                increment();
            }
        }
    }

    private static void increment() {
        counter.incrementAndGet();
    }

    public static void main(String[] args) throws InterruptedException {
        CounterThread counterThread = new CounterThread();
        counterThread.start();
        for (int i = 0; i < 1000; i++) {
            increment();
        }
        counterThread.join();
        System.out.println("Final number should be: 2000, but it's :" + counter.get());
    }
}
```

Now the `increment` method atomically increments the current value. The two threads will not interleave when trying to increment `counter` variable because it happens all in one operation.

## Locking

Locking is an interesting topic in concurrency. In Java, you could use either the *object’s intrinsic lock* or an *explicit lock* (implementation of the `Lock` interface). Both of these two locking strategies help us achieve thread-safety by synchronizing access to shared mutable data.

### Object’s Intrinsic Lock

Every object has an intrinsic lock associated with it. Any thread that wants access to a shared object has to acquire the intrinsic lock first so no other thread can access that object, and when the thread finishes with the lock it releases it so other threads may acquire it.

When you declare that a method as synchronized, the thread calling that method acquires the intrinsic lock for that method’s object and releases it when the method exits. In `CheckThenActRace` (see above) we had used the object’s intrinsic lock to synchronize access to shared mutable objects and also we can leverage it to establish happens-before relationships that are essential to visibility.

### Explicit Lock

Added in Java 5, the package `java.util.concurrent.locks` has multiple classes that implement the `Lock` interface. One of these classes is `ReentrantLock` which is a reentrant mutual exclusion Lock with the same basic behavior and semantics as the intrinsic lock accessed using `synchronized` methods and statements. Typical usage of this class:
```java
Lock lock = new ReentrantLock();

lock.lock();
try {
    //do critical section code. May throw exception
} finally {
    lock.unlock();
}
```

This trick makes sure that the Lock is unlocked in case an exception is thrown from the code in the critical section. if an exception occurred in the critical section and the `unlock()` wasn’t called from inside a `finally` block, the lock would remain locked forever, causing all threads trying to acquire the lock to hang indefinitely.

## Conclusion

In this article, we saw what is thread-safety is all about, how to write thread-safe code. We also talked about common pitfalls that may occur when developing concurrent programs and how to deal with them.

If you want to know more about the subject, I highly recommend reading “Java Concurrency In Practice” by Brian Goetz.

In the next article, there will be more Core Java. Stay tuned!
