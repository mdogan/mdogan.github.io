---
layout: post
title: How many bytes can I store here, sir?
published: true
tags: java memory jvm
comments: true
---

Every operating system has its own special procedure to access memory information and statistics. In Linux one can read `/proc` filesystem, in nearly every OS one can fork a subprocess to execute a system command to find out memory information or call a system dependent API.

But since *Java* has the motto of **write once run everywhere**, how can one access memory information in Java?

<!--excerpt-->

### Memory belonging to JVM process

Reading process memory usage and free process memory is already available using JDK public API. There are a few ways of achieving this;

* Using `java.lang.Runtime`:

```java
Runtime runtime = Runtime.getRuntime();
// maximum memory this JVM process can allocate
runtime.maxMemory();
// total allocated memory by JVM process
runtime.totalMemory();
// total free space in allocated memory
runtime.freeMemory();
```

* Using `java.lang.management.MemoryMXBean`:

```java
MemoryMXBean memoryMXBean = ManagementFactory.getMemoryMXBean();

MemoryUsage heapMemoryUsage = memoryMXBean.getHeapMemoryUsage();
// total allocated *heap* memory by JVM process
heapMemoryUsage.getCommitted();
// total used *heap* memory
heapMemoryUsage.getUsed();
// maximum *heap* memory this JVM process can allocate
heapMemoryUsage.getMax();

// also there's another *MemoryUsage* bean to access
// non-heap memory allocated by JVM (See DirectByteBuffer)
MemoryUsage nonHeapMemoryUsage = memoryMXBean.getNonHeapMemoryUsage();
...

```

### Physical And Swap Memory

There's no public API to access physical memory information (at least I don't know yet :). But you can guess that there should be an internal method hiding somewhere inside JDK. No need to go far, answer is in `com.sun.management` package.

`com.sun.management.OperatingSystemMXBean` bean has many useful methods to access physical and swap memory information (in contrast to *not-very-useful* `java.lang.management.OperatingSystemMXBean`).

```java
com.sun.management.OperatingSystemMXBean operatingSystemMXBean
        = (com.sun.management.OperatingSystemMXBean) ManagementFactory.getOperatingSystemMXBean();

// total physical memory available
operatingSystemMXBean.getTotalPhysicalMemorySize();
// free physical memory available
operatingSystemMXBean.getFreePhysicalMemorySize();
// total swap space available
operatingSystemMXBean.getTotalSwapSpaceSize();
// free swap space available
operatingSystemMXBean.getFreeSwapSpaceSize();

```

Biggest problem in your head should be what if this class is not available in my environment. First solution pops out from our minds is using reflection. But there's also a standard-ish way of accessing this data: *querying `MBeanServer`*

```java
public static long getTotalPhysicalMemorySize() {
    return queryPhysicalMemory("TotalPhysicalMemorySize");
}

public static long getFreePhysicalMemorySize() {
    return queryPhysicalMemory("FreePhysicalMemorySize");
}

public static long getTotalSwapSpaceSize() {
    return queryPhysicalMemory("TotalSwapSpaceSize");
}

public static long getFreeSwapSpaceSize() {
    return queryPhysicalMemory("FreeSwapSpaceSize");
}

private static long queryMBeanServer(String type) {
    MBeanServer mBean = ManagementFactory.getPlatformMBeanServer();
    try {
        ObjectName name = new ObjectName("java.lang", "type",
            "OperatingSystem");
        Object attribute = mBean.getAttribute(name, type);
        if (attribute != null) {
            return Long.parseLong(attribute.toString());
        }
    } catch (Exception ignored) {
        // means this information is not available
    }
    return -1L;
}
```

Go fill all emty space! It's yours!
