---
layout: post
title: Optimizing GC time by reducing Humongous allocations
---

Recently I was debugging into latency problems in a service and figured quite a few GC related issues on the way. This is a summary of the findings. 

**Background**

This was a Java 8 service running with ParallelGC, primarily consuming messages from an SQS, processing, and sending the response back to another SQS. The processing
involved a bunch of serialization/deserialization along with quite a lot of network calls.

Upon enabling G1GC for the same service, the GC activities spiked up by at least 15% and 

* **Step 1 - Enable GC logs**

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

* **Step 2 - Analyze the logs**

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

* **Step 3 - Tracing the humongous allocations**

Now that we know that humnogous object allocations are causing so much of trouble, the next step is to identify where exactly the allocations are happening. There are a few blog posts like [this](https://www.pingtimeout.fr/posts/2020-01-23-trace-humongous-allocations-with-bpf/) which explain a way to trace them. However I tried with [JavaFlightRecorder](https://docs.oracle.com/javacomponents/jmc-5-4/jfr-runtime-guide/about.htm) which worked quite smooth. 

Recording with Java Flight Recorder:

~~~
jdk1.8/bin/jcmd 2092 JFR.start settings=profile
2092:
Started recording 10. No limit specified, using maxsize=250MB as default.

Use jcmd 2092 JFR.dump name=10 filename=FILEPATH to copy recording data to file.
~~~

Once the output of JFR is available, we could analyze with [JDK Mission Control](https://www.oracle.com/java/technologies/jdk-mission-control.html).

The humongous allocations will be visible under `Event Browser -> Allocation Outside TLAB`.


* **Step 4 - Removing humongous allocations**

TBD

**Conclusion**

TBD

**References**