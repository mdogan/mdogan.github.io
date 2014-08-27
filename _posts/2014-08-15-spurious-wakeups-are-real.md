---
layout: post
title: Spurious wakeups are real!
published: true
tags: java, concurrency, hazelcast
comments: true
---

Most probably you heard *spurious wakeup* term many times, you read it in many API docs. But have you actually seen it in the wild? Is it real?

[Here is wikipedia definition](http://en.wikipedia.org/wiki/Spurious_wakeup):

> Spurious wakeup describes a complication in the use of condition variables as provided by certain multithreading APIs such as POSIX Threads and the Windows API.

> Even after a condition variable appears to have been signaled from a waiting thread's point of view, the condition that was awaited may still be false.


Simply, it means a thread can wakeup from its waiting state without being signaled or interrupted or timing out. To make things correct, awakened thread has to verify the condition that should have caused the thread to be awakened. And it must continue waiting if the condition is not satisfied.  


For example;

- using `Object.wait() / Object.notify()`:

```java

final long timeoutMillis = someTimeoutValue;
final Object mutex = new Object();
volatile Object response;

// Thread-1
synchronized (mutex) {
    long remainingWait = timeoutMillis;
    long waitStart = System.currentTimeMillis();
    // loop with condition is guard against spurious wakeup
    while (response == null && remainingWait > 0) {
        mutex.wait(remainingWait);
        remainingWait = timeoutMillis - (System.currentTimeMillis() - waitStart);
    }
}

// Thread-2
synchronized (mutex) {
    response = value;
    mutex.notify();
}

```

- using `Condition.await() / Condition.signal()`:

```java
final long timeoutMillis = someTimeoutValue;
final Lock lock = new ReentrantLock();
final Condition noResponse = lock.newCondition();
volatile Object response;

// Thread-1
lock.lock();
try {
    long remainingWait = timeoutMillis;
    long waitStart = System.currentTimeMillis();
    // loop with condition is guard against spurious wakeup
    while (response == null && remainingWait > 0) {
        noResponse.await(remainingWait, TimeUnit.MILLISECONDS);
        remainingWait = timeoutMillis - (System.currentTimeMillis() - waitStart);
    }
} finally {
    lock.unlock();
}

// Thread-2
lock.lock();
try {
    response == value;
    noResponse.signal();
} finally {
    lock.unlock();
}

```

Let's go back to actual question; is *sprurious wakeup* a theoretical incident or is it practically real?

I have to say that, it's (un)fortunately a real/living phenomenon that one must take care of. We (at [Hazelcast](http://hazelcast.org)) struggled/wrestled with an operation timeout bug for hours, days and weeks. And finally we found that the bug we were trying to hunt was actually just a *spurious wakeup*.

See [OperationTimeoutException](https://github.com/hazelcast/hazelcast/issues/2051) bug report and [the patch](https://github.com/hazelcast/hazelcast/pull/3207/files) fixing it.
