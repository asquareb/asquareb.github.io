---
layout: post
title: "HBase 1.0 - Offheap Cache Configuration"
date: 2016-03-12 10:55:14 -0500
comments: true
author: Biju Nair
sharing: true
footer: true
categories: [hbase, performance]
---
If you have been using HBase off-heap bucketcache, you may agree that configuration it is a [bit cumbersome](http://blog.asquareb.com/blog/2014/11/24/how-to-leverage-large-physical-memory-to-improve-hbase-read-performance/) to say the least. In version 1.0, the HBase development team [simplified the offheap cache configuration process](https://issues.apache.org/jira/browse/HBASE-11520). With the changes, the following are the steps to configure bucketCache for e.g. of size n GB.
<!--more-->
- Set the total off-heap memory size to be used by HBase to the environment variable ``HBASE_OFFHEAPSIZE``. One way to do this is to set it in ``hbase-env.sh``
```
export HBASE_OFFHEAPSIZE=(n+x)G
```
*Note* The "G" is for gigabytes. The value for x should be 1 to 2 GB and the value should be at the higher end for clusters handling high volume of transactions. 
- The other option to configure the total off-heap memory size is tp set the ``-XX:MaxDirectMemorySize`` JVM property. Again this value can be set in ``hbase-env.sh``
```
export HBASE_OPTS="$HBASE_OPTS -XX:MaxDirectMemorySize=(n+x)G"
```
If you are wondering what x GB is for, it is for Java direct memory used by hdfs client used by hbase to interact with the underlying hdfs filesystem. 
- Set the HBase property ``hbase.bucketcache.combinedcache.enabled`` to ``true`` so that on-heap cache will be used for index and bloomfilter blocks along with ``bucketcache`` for data blocks.
```
  <property>
    <name>hbase.bucketcache.combinedcache.enabled</name>
    <value>true</value>
  </property>
```
- Set the HBase property ``hbase.bucketcache.ioengine`` to ``offheap``.
```
  <property>
    <name>hbase.bucketcache.ioengine</name>
    <value>offheap</value>
  </property>
```
- Set the HBase property ``hbase.bucketcache.size`` to the size of memory which is allocated for bucket cache in mb.
```
  <property>
    <name>hbase.bucketcache.size</name>
    <value>n*1024</value>
  </property>
```
- Restart HBase server processess so that these changes can take in effect. 
```
