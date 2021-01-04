---
layout: post
title: "Design optimal HBase based applications"
date: 2015-02-07 21:53:29 -0500
comments: true
sharing: true
footer: true
author: Biju Nair
categories: [hbase, performance]
---
Leveraging the architecture of any data management system used in an application will yield the best performance and is the case with HBase. So before listing the key points to remember while designing HBase based applications, lets briefly look at the relevant architecture of HBase which will provide the required context.
<!-- more -->
  
<h2>Key value store</h2>
HBase at its core is a key value store that provides row and table abstractions for users. Some may wonder what that means and here is a quick high level explanation

- When a user writes a row to a table in HBase, value of columns are written separately in storage with a [key value](https://github.com/apache/hbase/blob/master/hbase-common/src/main/java/org/apache/hadoop/hbase/Cell.java) of row identifier, column family name, column name, timestamp and version number combined
- The key value pairs are stored in sorted order and that means columns which are part of a row in a HBase table are stored in a sequence in storage
- When a row is read by a HBase client, all the columns values are read individually and stitched together to present it back to the client as a single row

<h2>Write path</h2>
When a write action is taken on a HBase table row, the data is written first to a log (WAL) on disk for recovery. Then the data is written to in memory storage called ``memstore``. 

Data from each column family in a table utilizes separate memory area of memstore storage and when memstore thresholds are reached the memstores are flushed which creates immutable files on disk. Note that as with separate memory in memstore for different column families in a table, separate files are created in disk for each column family in a table.  

Since the disk files created are immutable, when an update request is made, data will be written with the latest timestamp which will be used by read requests to identify the latest data. Also for delete requests,a new row entry will be stored with a delete marker so that reads will be able to identify that the row is deleted.

<h2>Read path</h2>
When a read request is made, HBase identifies the files which stores the rows meeting the request criteria from the metadata it maintains. Each file includes a block index which will identify the block inside the file where the row can be found. HBase performs a scan of the block to retrieve all the key value pairs corresponding to the request and a copy gets stored in the ``block cache`` in memory before the row is returned back to the client. BlockCache stores the data in memory for further reads and the data in cache gets dropped using LRU algorithm when the cache gets filled. 

While scanning the file block, there is a possibility that there can be no data available which satisfies the read request. Also if a row spans multiple blocks HBase will scan multiple blocks to satisfy the read request.

Note that if there are more than one column family in a table and the read request involves data from more than one column family, HBase need to go through the process against multiple files on disk to retrieve data to satisfy the request. Also if the latest data for the row is available in memstore due to recent writes, HBase will merge data from the files in disk with the data in memstore before returning it to the client as request response.

<h2>Processing and storage maintenance</h2>
From HBase process perspective, table data is stored in ``regions`` which includes ``memstore`` and files on disk (``store files``). Regions are managed by region servers which handles all user requests. HBase master allocates ``regions`` to ``region servers`` during start up and it also maintains the table metadata.

In order to reduce the number of on-disk files which will also improve read performance, HBase performs regular minor compactions based on [configured thresholds](/blog/2015/01/01/configuration-parameters-that-can-influence-hbase-performance/) which will combine small files created by memstore flushes into a single large file. When the size of store (memstore + store file for a column family) grows beyond a certain limit, HBase by default will split the region into two and this automatic splitting process continues as more data is written to the table. The newly created region gets assigned to a new region server and hence the processing is distributed to additional node which improves performance and scalability.  

HBase by default also performs major compactions on all disk files in a region related to a table column family to create a single store file on disk which again improves read requests. During major compaction HBase also removes data for rows marked as deleted. But as one could guess there will be impact an on performance when major compaction occurs.

<h2>Application design considerations</h2>
Given this high level understanding of HBase which are relevant to anyone performing application design, the following are some of the points to take into account while designing an application using HBase to get optimal performance.

- Unlike traditional DBMS which can use data structures like indexes, HBase solely relies on table keys. So understanding the queries which are required to satisfy the user case and creating tables with the appropriate key is important.
- Table keys should also prevent data skew and processing skew i.e key should distribute data storage and processing to all region servers so that design can scale and perform well utilizing all the resources in the HBase cluster. 
- While blocks storing data relevant to a query is quickly identified ( ~O(3) ), scanning through the block to retrieve the data takes longer time ( O(n) n being the number of key values stored in a block) along with the time it takes to bring the block from disk into memory. It is important to create tables with optimal block size which will also help utilize cache optimally.
- If queries often retrieve data from more than one column family in a table which means HBase needs to deal with two sets for store files, it would be good to have related columns in a single column family which will reduce the number of disk files to deal with during reads.  
- Denormalize data since HBase doesn’t support multi-table joins. This will also help with data consistency since row level actions are atomic and HBase doesn’t support transactions where multiple actions can be committed or rolled back.  
- Pre-split the tables so that data is distributed to multiple region servers from the get go instead of starting with one region server and getting hammered. Also look at using other auto splitting options other than the default size based splitting of regions.
- Leverage versioning and TTL features to prune the amount of relevant data stored in the cluster.
- Unlike traditional databases, column names can be used to store the column values with dummy data for the actual value which will reduce the amount of data stored in the table and inturn also reduces block/cache usage.
- If there is a pre-defined maintenance window available to perform major compaction, disable major compactions so that spikes in low performance can be avoided.
- Enable bloom filters on all tables which will reduce the number of blocks read. There is storage overhead to enable bloom filter which may be a worthy price to pay for the performance gain.

These along with [JVM](/blog/2014/12/13/jvm-gc-settings-and-hbase/), [cache](/blog/2014/11/21/leverage-hbase-cache-and-improve-read-performance/) and HBase [configuration parameters](/blog/2015/01/01/configuration-parameters-that-can-influence-hbase-performance/) looked at earlier will help someone to get the best out of a HBase cluster. Also if HBase fits a use case and want to vet other options before making a decision Redis, Accumulo, Cassandra are some of the options to look into.

More notes on this category can be found [here](http://blog.asquareb.com/blog/categories/hbase/).
