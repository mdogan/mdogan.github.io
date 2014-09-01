---
layout: post
title: Give me some GC stats...
published: true
tags: java memory GC jvm
comments: true
---

While analysing application memory usage and inspecting allocation/garbage creation patterns, we generally need to know garbage collection count, time, rate etc. Although most of the profilers give this information out-of-the-box or JVM already has builtin flags to enable GC logging in many details, sometimes we want to access GC information programmatically.

Fortunately, there's a formal/public API in JDK to gather some useful information: `java.lang.management.GarbageCollectorMXBean`.

<!--excerpt-->

```java
GarbageCollectorMXBean gcMxBean = ...;

// name of the memory manager,
// aka name of the garbage collector
gcMxBean.getName();

// total collection count since process start
gcMxBean.getCollectionCount();

// total (approximate) collection elapsed time (ms)
gcMxBean.getCollectionTime()

```

Above code gives us only information about a garbage collector in system. But in a JVM there may be more than one garbage collector running for different memory regions, like eden space, tenured generation etc.

From this raw data, we need to gather some useful information. First we need to distinguish which memory manager belongs to which region. To be able to do this seperation, we need to know GC names in JVM. There are a few major JVM implementations, like Hotspot, JRockit or IBM J9. I'll do this for 3 most known Hotspot GC types;

```java
static final Set<String> YOUNG_GC = new HashSet<String>(3);
static final Set<String> OLD_GC = new HashSet<String>(3);

static {
    // young generation GC names
    YOUNG_GC.add("PS Scavenge");
    YOUNG_GC.add("ParNew");
    YOUNG_GC.add("G1 Young Generation");

    // old generation GC names
    OLD_GC.add("PS MarkSweep");
    OLD_GC.add("ConcurrentMarkSweep");
    OLD_GC.add("G1 Old Generation");
}

private static void printGCMetrics() {

    long minorCount = 0;
    long minorTime = 0;
    long majorCount = 0;
    long majorTime = 0;
    long unknownCount = 0;
    long unknownTime = 0;

    List<GarbageCollectorMXBean> mxBeans
        = ManagementFactory.getGarbageCollectorMXBeans();
    for (GarbageCollectorMXBean gc : mxBeans) {
        long count = gc.getCollectionCount();
        if (count >= 0) {
            if (YOUNG_GC.contains(gc.getName())) {
                minorCount += count;
                minorTime += gc.getCollectionTime();
            } else if (OLD_GC.contains(gc.getName())) {
                majorCount += count;
                majorTime += gc.getCollectionTime();
            } else {
                unknownCount += count;
                unknownTime += gc.getCollectionTime();
            }
        }
    }

    StringBuilder sb = new StringBuilder();
    sb.append("MinorGC -> Count: ").append(minorCount)
            .append(", Time (ms): ").append(minorTime)
            .append(", MajorGC -> Count: ").append(majorCount)
            .append(", Time (ms): ").append(majorTime);

    if (unknownCount > 0) {
        sb.append(", UnknownGC -> Count: ").append(unknownCount)
                .append(", Time (ms): ").append(unknownTime);
    }

    System.out.println(sb);
}
```

This is sample output from above method;

```
MinorGC -> Count: 3, Time (ms): 39, MajorGC -> Count: 0, Time (ms): 0
```

Now you need to find out why in the hell those precious CPU cycles are spent on doing GC.
