---
layout: post
title: "Configuration parameters that can influence HBase performance"
date: 2015-01-01 18:10:24 -0500
author: Biju Nair
comments: true
sharing: true
footer: true
categories: [hbase, performance]
---
Based on individual use case and HBase deployment the settings of the following parameter will vary. As with any performance improvement exercise all changes should be systematically tested for benefits.
<!--more-->
**HBase properties**

``hbase.regionserver.handler.count`` - controls the number of RPC listener threads on each region server. Increasing the listeners threads can improve performance but need to be balanced against the amount of resource available in the system since each thread consumes memory and CPU.

``hbase.ipc.server.callqueue.handler.factor`` - Determines the number of call queues interms of regionserver handler count. Dedicated queues for each RPC listener thread reduces contention.

``hbase.ipc.server.callqueue.read.ratio`` - Determines the number of call queues dedicated to read and write requests. If the work load is primarily read then increasing read queues to higher value value will help. By default there is no differentiation of queues as read or write.

``hbase.ipc.server.callqueue.scan.ratio`` - Determines the number of call queues allocated for gets and scan requests. If the work load is primarily gets and not scans increasing the get queues will help. By default all the read queues are used for both get and scan requests.

``hbase.regionserver.global.memstore.size`` - Ratio of JVM heap allocated for HBase memstore. The defaul is 40%. If the work load is primarily writes increasing this percentage will help and that means the percentage of heap allocated to blockcache need to be reduced since the total of the two can’t go beyond 80%.

``hbase.regionserver.region.split.policy`` - Policy which determines when a region to be split. There are a few options available and each of them can make a difference for different scenarios. 

``hbase.zookeeper.property.maxClientCnxns`` - Number of concurrent connections which can be made to a single member of ZK ensemble from a single client and this value should match the value in zoo.cfg. This need to be adjusted taking into consideration the expected number of HBase client connections.  

``hbase.client.write.buffer`` - Size of write buffer at client and server. Increased buffering will help but need to be balanced against memory available on the server side.

``hbase.client.scanner.caching`` - Number of rows fetched when calling next on a scanner if it is not served from client local memory.

``hbase.hregion.memstore.flush.size`` - Size when exceeded flushes the memstore and is checked at hbase.server.thread.wakefrequency intervals. Higher the number the 

``hbase.hregion.memstore.mslab.enabled`` - enabled by default to prevent heap fragmentation and better not to disable it.

``hbase.hregion.max.filesize`` - Sum of region HFiles when exceeded, region is split into two. It can help to have a higer value than the default 10 GB, if the workload is high volume/velocity write.

``hbase.hregion.majorcompaction`` - time between major compaction and a value of 0 disables it. It would be good to disable it and schedule it on a regular maintenance window to prevent performance issues.

``hfile.block.cache.size`` - percentage of JVM heap allocated for block cache. By default it is 40% but a higher percentage will help with read latencies if the primary workload is read. But any change need to be reflected in the memstore percentage.

Set ``hbase.ipc.client.tcpnodelay`` to true and is the case by default. 

Setting ``hbase.storescanner.parallel.seek.enable`` to true can help with read latencies

If data staleness is not a concern during reads for e.g. in stiuations where data is loaded nightly, enable [MTTR](https://issues.apache.org/jira/browse/HBASE-10070) so that read failures due to region failures can be prevented by HBase servicing the request from a (stale) copy of the data.

Enable **HDFS short circuit read** so that read latencies can be improved by reading data locally.

- Set ``dfs.block.local-path-access.user`` to hbase
- Set ``dfs.client.read.shortcircuit`` to true                
- Set ``hbase.regionserver.checksum.verify`` to true
                 
**On Linux OS:**

Set ``vm.swappiness`` to 0 or low value

As with any database based application, Even with a well tuned cluster with the best hardware, an application will not get the best performance if it is not designed properly.

More notes on this category can be found [here](http://blog.asquareb.com/blog/categories/hbase/)
