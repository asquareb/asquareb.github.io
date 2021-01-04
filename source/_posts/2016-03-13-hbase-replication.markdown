---
layout: post
title: "HBase Replication"
date: 2015-11-21 11:53:57 -0400
comments: true
author: Biju Nair
sharing: true
footer: true
categories: hbase
---
HBase supports inter-cluster data replication which can be used to propagate data to a secondary cluster/data center that can be accessed when primary cluster/data center is not available. The following are the high level steps to enable HBase inter-cluster replication. Note that HBase also supports region replication with in a cluster for read HA which is different from inter-cluster data replication.
<!--more-->
Set the ``hbase.replication`` property to ``true`` in hbase-site.xml of  the HBase cluster from which data need to be replicated from. This cluster is referred as the ``master`` going forward. By default the value of this property is "true".
{% codeblock %}
<property>
  <name>hbase.replication</name>
  <value>true</value>
</property>
{% endcodeblock %}
Create a HBase replication peer in the master HBase cluster using the information about the ZooKeeper quorum of the cluster to which data need to be replicated to. The cluster to which data will be replicated to will be referred as ``slave`` going forward.
{% codeblock %}
$ hbase shell
15/09/23 10:35:52 INFO Configuration.deprecation: hadoop.native.lib is deprecated. Instead, use io.native.lib.available
HBase Shell; enter 'help<RETURN>' for list of supported commands.
... 
hbase(main):001:0>add_peer 'HBASE_REPL_PEER','zk1,zk2,zk3:2181:/hbase'
0 row(s) in 0.3470 seconds
 
hbase(main):003:0> list_peers
PEER_ID CLUSTER_KEY STATE TABLE_CFS
HBASE_REPL_PEER zk1,zk2,zk3:2181:/hbase ENABLED
1 row(s) in 0.1520 seconds
{% endcodeblock %}
Once the replication peer is created and enabled, replication need to be enabled on HBase tables whose data need to be replicated from the master cluster by setting the "REPLICATION_SCOPE" attribute of the table to a non zero value. By default this value is set to "0". If the table is an existing table, altering the table to set the "REPLICATION_SCOPE" to a non zero value requires disabling and enabling the table and the following is an example of the steps where the existing table's name is "healthy". Note that a table with the same definition as the table being replicated (in this case "healthy") should be created in the slave cluster before the replication is enabled on master.
{% codeblock %}
hbase(main):006:0> describe 'healthy'
DESCRIPTION ENABLED
'healthy', {NAME => 'cf1', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER => 'ROW', REP true
LICATION_SCOPE => '0', VERSIONS => '1', COMPRESSION => 'NONE', MIN_VERSIONS => '0',
TTL => 'FOREVER', KEEP_DELETED_CELLS => 'false', BLOCKSIZE => '65536', IN_MEMORY =
> 'false', BLOCKCACHE => 'true'}
1 row(s) in 0.0430 seconds
 
hbase(main):007:0> disable 'healthy'
0 row(s) in 1.2940 seconds
 
hbase(main):015:0> alter 'healthy',{NAME => 'cf1', REPLICATION_SCOPE => '1'}
Updating all regions with the new schema...
1/1 regions updated.
Done.
0 row(s) in 1.1660 seconds
hbase(main):017:0> enable 'healthy'
0 row(s) in 0.2310 seconds
 
hbase(main):016:0> describe 'healthy'
DESCRIPTION ENABLED
'healthy', {NAME => 'cf1', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER => 'ROW', REP false
LICATION_SCOPE => '1', VERSIONS => '1', COMPRESSION => 'NONE', MIN_VERSIONS => '0',
TTL => 'FOREVER', KEEP_DELETED_CELLS => 'false', BLOCKSIZE => '65536', IN_MEMORY =
> 'false', BLOCKCACHE => 'true'}
1 row(s) in 0.0450 seconds
{% endcodeblock %}
Once replication is enabled replication related JMX stats are made available in all the region servers in the master cluster hosting the regions for which data replication is enabled. 
{% codeblock %}
"source.shippedKBs" : 3388621,
"source.logEditsFiltered" : 428856845,
"source.sizeOfLogQueue" : 0,
"source.HBASE_REPL_PEER.logEditsFiltered" : 378778983,
"source.HBASE_REPL_PEER-rs1,60200,1441910792839.shippedBatches" : 24,
"source.logReadInBytes" : 108666586903,
"source.HBASE_REPL_PEER.shippedOps" : 2097152,
"source.HBASE_REPL_PEER.logReadInBytes" : 96312291957,
"source.shippedOps" : 3098060,
"source.HBASE_REPL_PEER.shippedBatches" : 51,
"source.HBASE_REPL_PEER-rs1,60200,1441910792839.shippedKBs" : 1094799,
"source.logEditsRead" : 428859535,
"source.shippedBatches" : 75,
"source.HBASE_REPL_PEER-rs1,60200,1441910792839.logReadInBytes" : 12354294946,
"source.HBASE_REPL_PEER.shippedKBs" : 2293822,
"source.HBASE_REPL_PEER-rs1,60200,1441910792839.shippedOps" : 1000908,
"source.ageOfLastShippedOp" : 51072,
"source.HBASE_REPL_PEER.sizeOfLogQueue" : 0,
"source.HBASE_REPL_PEER.ageOfLastShippedOp" : 51072,
"source.HBASE_REPL_PEER.logEditsRead" : 378780481
{% endcodeblock %}
Note that these are statistics available in HBase version 0.98. Later versions may have additional statistics which can help with replication monitoring.
