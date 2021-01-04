---
layout: post
title: "JVM GC settings and HBase Performance"
date: 2014-12-13 19:13:31 -0500
author: Biju Nair
comments: true
categories: [HBase, Performance]
---
HBase being Java based and uses JVM heap differently than a typical Java process, Java garbage collection impacts the performance of HBase. The performance impact will be noticeable if JVM GC parameters are not set properly. 
Inorder to come up with optimal GC settings, it is good to understand GC process which will provide the required context. The following is a quick summary of GC process and the relevant parameters which influences its behavior. For anyone who likes to venture into the details of GC there is enough literature available on the net. 
<!-- more -->
- When JVM allocates heap it divides it into young and old generation space. The max heap space size can be set by the parameter ``-Xmx`` and the initial memory allocation for heap can be controlled using the parameter ``-Xms``. 

- All new java objects are allocated in the young generation space and the size is controlled by the parameter ``-Xmn`` which sets the initial and max size of young gen heap size.
	- If someone likes to have fine grained control over the initial allocation and the max size of young generation space, the ``-Xmns`` and ``-Xmnx`` parameters can be used.   

- As the young gen space gets full, JVM performs a minor (young gen) GC and any object which survives the minor GC gets moved to the heap space allocated for older gen.
	- The size of heap for old gen will be the total heap size (``-Xmx``) minus the size allocated for young gen (``-Xmn``).

- When the old gen heap space usage reaches a threshold, JVM performs major GC to recover space and depending on the collector used for GC performs compaction to reduce fragmentation. Major GC takes longer time relative to minor GC and JVM need to stop the application to complete it. Stopping of the application will show up as performance degradation of the application.  

The generational approach (i.e. young gen and old gen objects) to heap allocation and GC is based on the observation that most of the objects allocated by Java applications die quickly. Hence if the young gen is kept small, the minor GC will be quicker and the old gen heap space will not get polluted with transient objects. This will help efficient old gen GC and the overall stop time of the application required for GC. Inorder to determine which objects are non transient and can be promoted to old gen heap space JVM follows the following process.

- The young gen heap space is divided into `` eden space `` and 2 `` survivor spaces ``. The space allocation is controlled by the parameter ``-XX:SurvivorRatio`` and if it set to 6, the `` eden space `` will be of size `` Xmn * 6/8 `` and each `` survivor space `` will be of size `` Xmn * 1/8 ``.

- All new objects get created in `` eden space ``. When a minor GC happens objects which are still alive gets moved into one of the survivor space leaving the ``eden space`` second ``survivor space`` empty. When the next minor GC happens, objects alive in eden space along with the ones in survivor space gets copied to the second survivor space leaving the first survivor space empty. This process of copying live objects between the two survivor spaces happens until a threshold which determines that the objects are not transient since they survivied the number of minor GCs. The objects which survived are copied to old gen heap space. This process will reduce the number of objects copied to old gen space since many objects will get released during the process and the real survivors get copied to old gen space.

- Different garbage collectors are available to choose from based on the use case.
	- Serial GC enabled by the parameter `` -XX:+UseSerialGC `` uses single thread to perform GC and requires long application pause time to complete GC
	- Parallel GC enabled by the parameter `` -XX:+UseParallelGC `` runs multiple threads to perform GC which reduces application pause time relative to serial GC but requires more CPU
	- Concurrent mark and sweep (CMS) enabled by the parameter `` -XX:+UseConcMarkSweepGC `` runs some of the steps in the GC process (specifically marking the objects as garbage for collection) in parallel to running the application which reduces the application stop time
	- Garbage First enabed by the parameter`` –XX:+UseG1GC `` which is similar to CMS but the memory allocation model and the algoritm is focussed on minimizing the number of old gen GC.

As you can guess there is more than what is written here about the collectors and there is enough literature to understand them in detail.

- Setting aside `` Garbage First `` collector for the time being, for HBase which is deployed mostly for low latency, CMS is the best option for the old gen GC and parallel GC of the young gen GC assuming that there are more than one CPU cores are available.
	- Use `` -XX:+UseConcMarkSweepGC `` to use CMS
	- Use `` -XX:+UseParNewGC `` to use parallel GC for young gen GC and `` -XX:ParallelGCThreads=nm `` to define the number of parallel threads and the value should be less than the number of CPU cores available.

- Since young GCs are frequent relative to old gen GCs, requires the application to stop and the GC time depends on the size of the young gen space irrespective of the collector used, the young gen space need to be kept small so that the application stop time is minimal. A rough estimation on the size of young gen (`` -Xmn ``) can be 100 m * number of cores or number of parallel GC threads allocated for young gen GC.

- Apart from creating typical Java objects, HBase also uses the heap to explicitly allocate and service queries. During read operation HBase creates `` blockcache `` objects of size 64K and on write creates `` memstore `` objects of size  2K. Most of these objects will be survivor objects since HBase determines when they get release to become garbage. In the case of `` blockcache `` it is the LRU algorithm used to release objects and in case of `` memstore `` objects are released during flushes. What this means is that these object can be moved to old gen from young gen quickly instead of JVM determining the threshold by copying these objects between the two survivor spaces before moving the objects to the old gen space. This can be controlled by setting the parameter `` -XX:MaxTenuringThreshold=n ``. In tests setting it to 3 or 4 has provided good balance. But it can be fine tuned by printing the tenuring distribution on existing deployments by setting the parameter ``-XX:+PrintTenuringDistribution`` which will provide details about the number of times JVM copied objects between the survivor spaces before moving the objects to old gen space.

- Since HBase creates many survivor objects (`` blockcache `` and `` memstore `` objects) it would help if the ratio of survivor space is increased relative to the eden space. By default eden space is 6/8 of young gen space leaving each survivor space 1/8th of young space. This can be changed by setting ``  -XX:SurvivorRatio `` and a value of 4 will increase the survivor space to 1/6 of young space. By increasing survivor space and reducing the tenuring threshold more `` blockcache `` and `` memstore `` objects will be moved to old gen space during minor (young gen) GC.

- The threshold which triggers the CMS GC on old gen heap is determined by the parameter `` -XX:CMSInitiatingOccupancyFraction=nn ``. By default it is set to 69 which is the percentage of the old gen space when gets occupied the CMS GC gets triggered. After the initial GC, JVM determines automatically when to trigger the GC based on its observation. But `` -XX:CMSInitiatingOccupancyFraction `` plays a significant role i.e. even if there are no dead objects GC gets triggered if the old gen objects occupies more than the occupation fraction. This is key parameter since `` blockcache `` and `` memstore `` objects which are live objects can trigger old gen GC if the occupation fraction is not set a value higher than `` higher of ( blockcache size / size of old gen space and memstore size / size of old gen space) ``. Also we want the occupation fraction to be the only factor to be used to trigger all old gen GC instead of JVM determining it dynamicallyand this can be controlled by using the parameter ``  -XX:UseCMSInitiatingOccupancyOnly ``.

- [Young gen GC time also depends on the total heap size](http://blog.ragozin.info/2012/03/secret-hotspot-option-improving-gc.html) since it need to determine the live objects in young space which also involves looking for references from the objects from the old gen to the objects in young gen space. The `` -XX:ParGCCardsPerStrideChunk `` parameter controls the amount of old gen space allocated to GC threads to identify the dirty pages and by default it is set to 512 byte pages which is too small for large heap sizes typically used in HBase installations. Setting a higher value of 16 or 32 K will reduce the young gen GC time considerably.

- In order to understand the current GC behavior which will help with fine tuning the GC settings, use the options ``-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -XX:+PrintTenuringDistribution -XX:+PrintGCApplicationStoppedTime  -Xloggc:/var/log/hbase/gc/gc.log-$$-$(hostname)-$(date +'%Y%m%d%H%M').log `` which will generate information which can be used for tuning..

G1 collector is similar to CMS collector but raises a [question about collector improvements for general Java based applications and specialized systems like HBase using Java](/blog/2014/12/18/garbage-first-g1-gc-and-hbase/).

More notes on this category can be found [here](http://blog.asquareb.com/blog/categories/hbase/). 
