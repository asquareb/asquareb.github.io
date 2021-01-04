---
layout: post
title: "Garbage First (G1) GC and HBase"
date: 2014-12-18 22:31:37 -0500
author: Biju Nair
comments: true
sharing: true
footer: true
categories: [hbase, performance]
---
G1 the lastest Java garbage collector is a generational collector like [CMS](/blog/2014/12/13/jvm-gc-settings-and-hbase/). The key differences between G1 and CMS collectors are

- In CMS the young and old generation heap space are physically differentiated. But in the case of G1 they are logical. This is achieved by dividing the total heap space into number of ``regions`` (2000) and the regions are identified whether they belong to young or old space.
- Compaction of the heap space is done as part of the collection process which was not the case in CMS.
- G1 collector tries to meet user specified target for application pause time during the collection process.
<!-- more -->
As with [CMS](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/G1GettingStarted/index.html), heap objects are created in eden space first. Then objects are copied to survivor space and objects which survive the aging threshold during minor GC gets moved into old gen space. 

When GC gets triggered, G1 can identify liveliness of objects in the various ``regions`` in a particular space (young or old). In order to fully utilize heap space and at the same time meet the acceptable application pause threshold, G1 collects the regions with lowest liveliness (hence more garbage) since it will be fast in releasing more heap space. Also during GC, live objects remaining in a ``region`` gets copied to another region with enough space to accomodate the objects. Freeing up regions results due to this copy process results in better compaction of heap space. Since GC pause times are a major component to HBase latency and latency fluctuations, any reduction in the pause time as with G1 collector will improve HBase latency.

One interesting question which pops up is whether the compaction step in the G1 process is necessary for a system like HBase. Compaction which involves copying of objects between regions which requires resetting of references will be the costliest operation in the G1 collection process. 

In the case of HBase most of the objects which will survive and get moved to old gen heap space will be memstore or blockcache objects. They are of fixed sizes (2K and 64K respectively) and hence any new objects which gets moved to old space will be able to fit into a slot which got released earlier by HBase. So the only advantage of compaction of heap for a system like HBase is to be able to free GC regions quickly. But the time to perform G1 GC that meets user specified threshold includes compaction and freeing of regions, it is hard to imagine the advantage of G1 GC over CMS where there is no compaction.

While G1 collector is set to replace CMS in the future and will be improved upon to better the performance of Java based systems in general, for systems like HBase which has a prescribed heap usage and operational pattern the improvements may not always be beneficial. One of the options to reduce the impact of GC on HBase subsystems, is to use more of offheap memory like [BucketCache](/blog/2014/11/24/how-to-leverage-large-physical-memory-to-improve-hbase-read-performance/) since objects in offheap memory is not managed by GC and it is left to applications to manage it. This sounds more like the earlier database systems where services were inbuilt instead of relying on the [OS based services which are general purpose](http://www.eecs.berkeley.edu/~prabal/resources/osprelim/Sto81.pdf).

More notes on this category can be found [here](http://blog.asquareb.com/blog/categories/hbase/).
