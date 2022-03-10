---
layout: post
title: Optimizing GC time by reducing Humongous allocations
---

Recently I was debugging into latency problems in a service and figured quite a few GC related issues on the way. This is a summary of the findings. 

**Background**

This was a Java 8 service running with ParallelGC, primarily consuming messages from an SQS, processing, and sending the response back to another SQS. The processing
involved a bunch of serialization/deserialization along with quite a lot of network calls.

Upon enabling G1GC for the same service, the GC activities spiked up by at least 15% and 

* Step 1 - Enable GC logs

Enabling GC logs provides more details on whats going on in the service.

~~~
-XX:+PrintGCDetails 
-XX:+PrintGCDateStamps
-XX:+PrintGCApplicationStoppedTime
-XX:+PrintGCApplicationConcurrentTime
-XX:+PrintTenuringDistribution
-XX:+PrintAdaptiveSizePolicy
-XX:+UseStringDeduplication
-XX:+PrintStringDeduplicationStatistics
~~~

* Step 2 - Analyze the logs

After enabling detailed GC activity logging, we could see that a lot of GC pauses were ocuring due to `Humongous Object` allocations. [Humongous Objects](https://docs.oracle.com/javase/10/gctuning/garbage-first-garbage-collector.htm#JSGCT-GUID-D74F3CC7-CC9F-45B5-B03D-510AEEAC2DAC) mean, memory
chunks of size more than 50% of the G1 block size. The G1 block size depends on the heap memory size and for us it was 4M. So, essentially, any memory allocation more than
2M was considered as `Humongous` and too many of those was creating high GC activities and pause times.

The logs were analyzed using [GCEasy](https://gceasy.io/). A really good handy tool to quickly analyze the GC logs.

Sample logs:

~~~
2021-11-26T09:20:57.159+0000: 87437.995: [GC pause (G1 Humongous Allocation) (young) (initial-mark)
2021-11-26T09:21:01.763+0000: 87442.598: [GC pause (G1 Humongous Allocation) (young)
2021-11-26T09:21:04.983+0000: 87445.819: [GC pause (G1 Humongous Allocation) (young)
~~~

* Step 3 - Tracing the humongous allocations

TBD

* Step 4 - Removing humongous allocations

TBD

**Conclusion**

TBD
