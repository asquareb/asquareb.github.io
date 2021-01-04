---
layout: post
title: "Java Direct ByteBuffer Performance Advantages and Considerations"
date: 2015-06-05 21:38:40 -0400
comments: true
author: Biju Nair
sharing: true
footer: true
categories: [Performance, Programming] 
---

During execution, objects/variables created by Java programs gets their space allocated in the JVM heap memory. The total amount of heap memory available for a JVM is determined by the value set to -Xmx parameter when starting the Java process. When object allocated is released by the Java program, the corresponding memory is made available for later use by the JVM garbage collection (GC) process. 

The GC process gets invoked typically when the amount of free memory in the JVM falls below a certain threshold. At a very high level, the GC process involves identification of objects which are not used any more i.e. not referenced anymore, releasing the memory and compacting the memory to reduce memory fragmentation. Readers who are interested in understanding the details of GC process can find it [here](http://blog.asquareb.com/blog/2014/12/13/jvm-gc-settings-and-hbase). As one can imagine, the time it takes to complete the GC process will increase with the increase in size of the Java heap memory since it takes more time to identify the objects which can be released and also to perform compaction. 
<!--more-->
If a Java application requires large memory (in GBs), the time it takes to complete the GC process will be detrimental to its performance. If the application is performance sensitive, then large heap memory size can adversely impact its performance. In order to mitigate this, one can try to use memory outside Java heap and hence reduce the Java heap memory use and its size. This can be done using the Java [ByteBuffer](http://docs.oracle.com/javase/7/docs/api/java/nio/ByteBuffer.html) class which provides the option to allocate ByteBuffers outside JVM heap using [allocateDirect](http://docs.oracle.com/javase/7/docs/api/java/nio/ByteBuffer.html#allocateDirect(int)) method.

The allocateDirect method allocates memory of requested size (in bytes) on memory outside the JVM heap (off-heap) and provides the object reference to the application with the starting offset of 0. The application can then use the reference to store and retrieve data into the off-heap memory. When the garbage collection runs, it doesn't have to take into account the memory allocated off-heap to identify memory not being used or perform memory compaction which in turn reduce the time to complete GC.

While the time to complete GC can be reduced when large memory is used in a Java process by using off-heap memory, there are other overheads which need to be taken into consideration before using it. Allocation of off-heap memory will take more time than the on-heap memory since the JVM need to make native calls to get the memory allocated. Also when the off-heap memory is not used anymore by the application, during GC process, the JVM need to make native calls to free the off-heap memory in addition to releasing the memory used by the object reference in on-heap memory. Also as per the API documentation, the JVM will make the best effort to not to use any on-heap memory as a intermediate step to store and retrieve data to/from the off-heap memory. In order to compensate these additional overheads and at the same time take advantage of using large memory without the penalty of increased GC time, it is best to use off-heap memory for large objects which doesn't get released often.

When a JVM is brought up to run a Java process, the total memory which can be used for off-heap memory can be specified using the JVM parameter ``-XX:MaxDirectMemorySize`` parameter. If the parameter is not set explicitly, the value is set to the free memory available in the system at the start of the process using [VM.maxDirectMemory()](https://github.com/openjdk-mirror/jdk/blob/icedtea/jdk7/master/src/share/classes/sun/misc/VM.java#L193) method call. When off-heap memory allocation is made, the JVM keeps track of the total memory used so far. When a new off-heap memory allocation request is made the JVM checks whether the sum of the requested memory size and the total memory allocated so far is greater than the available direct memory size set at the start of the Java process. If the sum exceeds the available memory, an explicit GC system call is made by the memory allocator and then the process thread sleeps for [100ms](https://github.com/openjdk-mirror/jdk/blob/icedtea/jdk7/master/src/share/classes/java/nio/Bits.java#L651) for the GC call to complete. After 100ms the allocator checks again to see whether there is enough space to satisfy the new memory allocation request before raising an out of memory exception.

Few things to note about this allocation process. 

- For performance sensitive applications the explicit GC and the non-tunable sleep time in the allocation logic when there is not enough memory can be a large overhead. 

- The second item to note is that a GC call means best effort will be made by the JVM to schedule one and doesn't guarantee that one will be run immediately. So there can be situations when the Java process will fail with OOM error even when there is enough memory to be freed to accommodate new memory allocation request since a GC is not run immediately. 

- Third, the thread sleep time of 100ms may not be sufficient in certain situations for the GC to complete and release unused memory to satisfy new memory allocation request. If any one is surprised that the 100ms is more than sufficient for a GC to complete, we came across the situation where trying to allocate 1 GB chunks of off heap space using a simple for loop failing on Ubuntu 12.04 LTS with OOM while the same runs fine on Redhat Linux machine which had a relatively less powerful hardware. With the current API this sleep time can't be adjusted and hence the application may have to perform additional sleep to make sure that there is no memory to use. 

- The last item of interest is the total memory available for use to allocate off-heap memory. This value is set at the start of the process either manually or by the VM. When set manually, the JVM doesn't verify whether there is enough free memory available on the system. Even if the value is set automatically by the JVM, the available memory on the system can be lower during the process execution since memory usage of other processes in the system can change as time goes by which can result in the Java process failing due to unexpected exception in memory allocation. So it is important to make sure that the memory of size set in ``-XX:MaxDirectMemorySize`` is available for the Java process to use so that the failures doesn't happen. 

There are few options the JVM can do to prevent allocation related exceptions which would require changes to the JVM code.

- Verify the system free memory to make sure that it is greater than or equal to what is set by the users when the JVM is brought up and also during the process execution. This will require native system calls and may have a pronounced impact on the performance of the Java process. One way to mitigate is to provide an JVM option for the users to set if they need this strict condition checking.

- Instead of invoking a GC call when all the memory is used, it would be better to have a configurable parameter to set direct memory used threshold to make the GC call. This should be a fairly simple change to the JVM code.

- Calculate the sleep time after the GC process taking into consideration all the factors which impacts GC time. This will be complex and will not be of less importance if the previous suggestion is implemented in the JDK code.
