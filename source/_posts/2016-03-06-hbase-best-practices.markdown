---
layout: post
title: "HBase - Best Practices"
date: 2015-09-26 19:06:10 -0500
author: Biju Nair
comments: true
sharing: true
footer: true
categories: [hbase]
---
**Application Development**

**Connection Object Reuse**

Creating connections to a server component from an application is a heavy weight operation and it is much pronounced when connecting to a database server. That being the reason database connection pooling is used to reuse connection objects and HBase is no exception. In HBase, data from meta table that stores details about region servers that can serve data for specific key ranges gets cached at the individual connection level that makes HBase connections much heavier. So if there are region movements for balancing or if a region server fails, the meta data need to be refreshed for each connection object which is a performance overhead. For these reasons, applications need to try to reuse connection objects created.
<!--more-->
The following code snippet shows how to create a HBase connection object in a Java application using HBase.

```
Configuration conf = HBaseConfiguration.create();
conf.set("hbase.zookeeper.quorum", "localhost");
HConnection conn = HConnectionManager.createConnection(conf);
```
If the application is multi-threaded, then it need to reuse the connection object to perform any data manipulation operations on tables. This can be achieved by individual threads creating the HTable object using the getTable(TableName) method of the HConnection object.
Once the data manipulation operations are complete each thread should close corresponding HTable but not the HConnection object so that it can be reused by other threads. 

**Pre-split Table**

In order to prevent skews in processing of queries and to distribute query processing work load across all the nodes in the cluster, it is a good practice to create tables which is pre-split. The key is to identify the split point so that the data will be distributed across all the nodes in the cluster. Once the split point is identified the table can be created pre-split using HBase shell and the following is an example of a table with 3 split points. 

```
hbase(main):001:0> create 'split_table', 'cf1', {SPLITS => ['a','g','o']}
0 row(s) in 2.5540 seconds
=> Hbase::Table - split_table
```

During start of development, when the split points in the data are not clear but if some one still want to pre-split the table, HBase provides a utility program which can split the table and uniformly distribute the data. The following is an example which creates a table with 10 splits and columnfamily 'cf1'.

```
$ hbase org.apache.hadoop.hbase.util.RegionSplitter 'split_table2' UniformSplit -c 10 -f 'cf1'
```

If you are creating tables programmatically using Java APIs, the following code snippet shows how to pre-split the table during creation

```
// Instantiating configuration class
Configuration conf = HBaseConfiguration.create();
System.out.println(conf.get("hbase.zookeeper.quorum"));
conf.set("hbase.zookeeper.quorum", "localhost");
conf.setInt("hbase.zookeeper.property.clientPort",2181);
// Creating a connection
Connection conn = ConnectionFactory.createConnection(conf);
// Instantiating admin class
Admin admin = conn.getAdmin();
// Instantiating table descriptor class
HTableDescriptor tableDescriptor = new
HTableDescriptor(TableName.valueOf("emp5"));
byte[][] splitKeys = ...;
// Adding column families to table descriptor
tableDescriptor.addFamily(new HColumnDescriptor("market"));
tableDescriptor.addFamily(new HColumnDescriptor("corp"));
// Create and pre-split the table through admin
admin.createTable(tableDescriptor, splitKeys);
System.out.println(" Table created ");
admin.close();
conn.close();
```

For further reading and understanding the details about HBase table splitting and merging refer [this blog post](http://hortonworks.com/blog/apache-hbase-region-splitting-and-merging). 
