---
layout: post
title: "Leverage HBase Cache and Improve Read Performance"
date: 2014-11-21 16:51:39 -0500
author: Biju Nair
comments: true
categories: [hbase, performance]
---
As with any database management system, proper utilization of caches will improve the query performance in HBase. If someone is looking to optimize caching, it is good to understand the HBase data structures since optimization will vary with each use case. The following is a simplistic view of HBase read write path of HBase and the participating components.
  
HBase has two in-memory data structures (memstore, blockcache) and two on disk data structures (WAL, HFile). During data write, HBase writes data into WAL (Write Ahead Log) on disk and also to memstore in memory. When a memstore utilization threshold is reached data is flushed into HFiles on disk. 
<!-- more -->
During read, data is read from HFile blocks into blockcache in memory and if required merge latest data in memstore before sending back the data to the client. If data is already in blockcache, it is not read from HFile i.e. from disk which improves read performance by avoiding disk reads. If you are someone who had been working with DBMS, these should be very familiar. But this is a simplistic view hiding details about the data structures which differ with each DBMS.

Since blockcache helps with improving query performance the first step is to size it appropriately. The total size of memstore and blockcache the two in memory data structures should not exceed 80% of the total HBase region server Java heap size. This is a hard limit in HBase to prevent over use of Java heap which can cause disruptions to JVM functions like GC. ** hbase.regionserver.global.memstore.size ** and ** hfile.block.cache.size ** hbase-site.xml properties sets the percentage of JVM heap which need to be allocated to memstore and blockcache respectively. By default both are set to 0.4 (40 %). If an application is read intensive for e.g. repeatedly reading same data and data to read gets written in batch, then increasing the percentage allocation of heap to blockcache will improve read performance since more data can be stored in cache. Note that increasing the blockcache size must by followed decreasing the memstore heap allocation so that the combined total does not exceed 80% of JVM heap.

|  Property Name                         | Original |  Changed |
|:---------------------------------------|:--------:|:--------:|
| hfile.block.cache.size                 |   0.4    |    0.6   
| hbase.regionserver.global.memstore.size|   0.4    |    0.2   


By allocating more heap to block cache, more data can be in memory which in turn improves read latencies. Another option to get more data into memory is to reduce the block size of the data stored in disk. Why?

When a row is requested by client, the block corresponding to where the row is stored on disk (store file) is read into memory (cache) before sending it back the requested data to the client. So by decreasing the block size more relevant data can be stored in cache which can improve read performance. Reducing the block size is not good for all scenarios. But here is an example where reducing the block size will help. 

If the use case is to read the latest data about tickers stored in a row and the data size is 5 K in size. Since the probability that users will be reading same (popular) tickers repeatedly is high, the relevant data which we need to store in memory are the popular ticker data. But if the block size is 64 K which is the default and the row size is 5 K, we can end up wasting up to 59 K of cache which is almost 93% of non relevant data stored in memory which can cause relevant data to be moved out of cache when it gets full. In this scenario reducing the block size to 8 K will improve considerably the amount of relevant data stored in cache and will improve the query performance. One drawback of reducing the blocksize is the increase in index and meta data stored in store files  and cache which may be small price to pay for the performance gain.

The following is an example of creating a table with 16 K block size column family through HBase shell
```
create 'happydays',{NAME => 'cf', COMPRESSION => 'SNAPPY',BLOCKSIZE=>16384},{NUMREGIONS=>24, SPLITALGO => 'UniformSplit'}
```
Note that the compression option helps in reducing the storage size on disk, but doesn't have impact on the cache size used since the data is decompressed before it is stored on memory.

Given that physical servers currently come with large memory (say 256 GB), can we allocate HBase heapsizes of say 96 GB so that we can store large amount of data in cache? The answer is no. We will look at the reason and an option to leverage the available physical memory [next](/blog/2014/11/24/how-to-leverage-large-physical-memory-to-improve-hbase-read-performance/).

More notes on this category can be found [here](http://blog.asquareb.com/blog/categories/hbase/).
      
For any one interested in visuals, the following presentation may help.

{% slideshare 37639243 %}
