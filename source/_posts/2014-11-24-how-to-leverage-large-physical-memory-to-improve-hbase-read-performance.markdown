---
layout: post
title: "Leverage large physical memory to improve HBase read performance"
date: 2014-11-24 08:29:34 -0500
author: Biju Nair
comments: true
categories: [hbase, performance] 
---
HBase uses [block cache](/blog/2014/11/21/leverage-hbase-cache-and-improve-read-performance/) to store data read from disks in memory so that data referenced repeatedly are serviced with out disk reads. Block cache uses the HBase JVM heap to store cache data and that means any factor which adversely impacts JVM GC process will impact cache and hence query performance.It is commonly known that large heap sizes of say more than 16 GB will make "stop the world" instances of GC run very slow and the time taken to complete can even make the HBase region server to be marked as dead.
<!-- more -->
Given that physical servers currently deployed has large memories like 128 GB or 256 GB, the 16 GB JVM heap limitation will result in inefficient server utilization. Apart from inefficient utilization, the ability to utilize the large memory to store data in HBase cache which will considerably improve query performance will be missed.

To mitigate this drawback, HBase includes another in-memory data structure called BucketCache. Unlike block cache which utilizes the JVM heap, bucket cache utilizes offheap memory created using Java `` ByteBuffer ``. Bucket cache is not enabled by default but once enabled large physical memory can be allocated to bucket cache since it is not impacted by JVM GC as with block cache. By being able to allocate large memory to HBase for caching, large volume of data can be stored in memory which will improve query latencies. In tests we were able to allocate BucketCache of upto 91 GB and data loaded into it with no issues.

When bucket cache is enabled in HBase, it will act as L2 (level 2 similar to processor cache hierarchy) cache and the default block cache will act as L1 cache. What that means is data need to be copied from L2 cache into L1 cache for HBase to service the query request. So on a read request when bucket cache is enabled, data block will be copied from disk onto bucket cache and then onto the block cache before served to the client making the request. On further reads of the same data, it will either be served directly from block cache or copied from  bucket cache into block cache before serving the client request avoiding disk reads. So if there is a 10 node cluster with 128 GB of RAM each, it is possible to have almost 1 TB of data stored in HBase cache with bucket cache enabled and not get impacted by JVM GC which is not the case using just the default block cache.

Inorder to enable bucket cache, changes need to be made to HBase JVM parameters in `` hbase-env.sh `` file and new HBase properties need to be set in `` hbase-site.xml `` file. Following summarizes the properties and the calculation on how to set the properties.

Let us assume the following

| Memory type                            | Mnemonic |
|:---------------------------------------|:--------:|
| Physical memory available for HBase RS |  Tot     
| Memory size for memstore               |  Msz     
| Memory for block cache (L1) cache      |  L1Sz    
| Memory for JVM related components      |  JHSz    

The following are the properties and the values to be set on each HBase region servers.

| Location       |  Name                                        | Mnemonic    | Value            |
|:---------------|:---------------------------------------------|:-----------:|:-----------------|
| hbase-env.sh   | XX:MaxDirectMemorySize                       |   DMem      | Tot-Msz-L1Sz-JHSz
| hbase-env.sh   | Xmx                                          |   Xmx       | MSz+L1Sz+JHSz 
| hbase-site.xml | hbase.regionserver.global.memstore.upperLimit|   ULim      | MSz/Xmx
| hbase-site.xml | hfile.block.cache.size                       |   blksz     | 0.8-ULim
| hbase-site.xml | hbase.bucketcache.size                       |   bucsz     | DMem+(blksz*Xmx)
| hbase-site.xml | hbase.bucketcache.percentage.in.combinedcache|   ccsz      | 1-((blksz*Xmx)/bucsz))
| hbase-site.xml | hbase.bucketcache.ioengine                   |             | offheap or file:/localfile

The following is an example

|      Item                                    |  Value     |
|:---------------------------------------------|:-----------|
| Physical memory available for HBase RS       |  96000 MB  
| Memory size for memstore                     |  2000 MB   
| Memory for block cache (L1) cache            |  2000 MB   
| Memory for JVM related components            |  1000 MB   
| hbase.regionserver.global.memstore.upperLimit|  91000     
| Xmx                                          |  5000      
| hbase.regionserver.global.memstore.upperLimit|   0.4      
| hfile.block.cache.size                       |   0.4      
| hbase.bucketcache.size                       |  93000     
| hbase.bucketcache.percentage.in.combinedcache|  0.97849   
| hbase.bucketcache.ioengine                   |  offheap   

Apart from acting as L1 cache the block cache should be able to store block indexes and bloom filters and so should be sized accordingly. The size of block index and bloom filters can be easily identified by looking at the HBase store file details displayed on the HBase master URL. Changes mentioned need to be made to all the HBase region server nodes for bucket cache to be enabled and all the region server nodes need to be restarted for the changes to take in effect. To ease these calculations use the simple python script available [here](https://github.com/bijugs/simple-scripts/blob/master/hbase-bucketcache.py).

If using more than 22 GB of offheap memory, it is better to mount the RAM as a [tempfs](http://en.wikipedia.org/wiki/Tmpfs) and use the `` file:/localfile `` value for the  `` hbase.bucketcache.ioengine `` hbase-site.xml property. This is due to the existing open issue [HBASE-10643](https://issues.apache.org/jira/browse/HBASE-10643).

More notes on this category can be found [here](http://blog.asquareb.com/blog/categories/hbase/).

For any one interested in visuals, the following presentation may help.

{% slideshare 37639243 %}
