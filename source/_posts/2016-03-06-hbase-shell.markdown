---
layout: post
title: "HBase Shell"
date: 2015-07-05 18:05:25 -0500
comments: true
sharing: true
footer: true
categories: [hbase]
---
HBase shell is a Ruby based shell which can be used to interact with HBase cluster and perform data definition and data manipulation tasks. The shell is made available as part of the HBase code base and by default gets installed on all the nodes of the HBase cluster. In order to access the shell, HBase software need to be installed on the machine from where the shell is invoked and the PATH environment variable updated with the directory where the hbase shell program is stored. Commonly users and administrators access the shell through one of the nodes in the HBase cluster. Following is a quick introduction to frequently used hbase shell commands.
HBase shell is invoked using the hbase shell command which in turn provides the user with a command prompt to enter the commands supported by the particular version of HBase.
<!--more-->
```

$ hbase shell
...
hbase(main):001:0> help
...

COMMAND GROUPS:
   Group name: general
   Commands: status, table_help, version, whoami
   Group name: ddl
   Commands: alter, alter_async, alter_status, create, describe, disable, disable_all, drop, drop_all, enable, enable_all, exists, get_table, is_disabled, is_enabled, list, show_filters
```

Further details on a particular command can be found using help 'command-of-interest'. The following is the partial output from help on create command help create

```
hbase(main):002:0> help 'create'
Creates a table. Pass a table name, and a set of column family
specifications (at least one), and, optionally, table configuration.
Column specification can be a simple string (name), or a dictionary
(dictionaries are described below in main help output), necessarily
including NAME attribute.
Examples:
Create a table with namespace=ns1 and table qualifier=t1
   hbase> create 'ns1:t1', {NAME => 'f1', VERSIONS => 5}
Create a table with namespace=default and table qualifier=t1
   hbase> create 't1', {NAME => 'f1'}, {NAME => 'f2'}, {NAME => 'f3'}
   hbase> # The above in shorthand would be the following:
As one would expect creating a table is using create command. The following is an example of creating a table TestTable with column family cf. The column family is of BLOCKSIZE 16K, uses SNAPPY COMPRESSION and the BLOOMFILTER attribute set to ROWCOL. 
create Table
hbase(main):014:0> create 'TestTable', {NAME => 'cf', BLOCKSIZE => 16384, BLOOMFILTER => 'ROWCOL', COMPRESSION => 'SNAPPY'}
0 row(s) in 0.5020 seconds
```

Note that by default a table is created in the default namespace if the namespace is not specified during table creation. If a namespace is specified during table creation, the namespace should already exist and it can be created using create namespace command.
To view the properties of a table describe command can be used.
```
hbase(main):017:0> describe 'TestTable'
DESCRIPTION                                                                                               	ENABLED
  'TestTable1', {NAME => 'cf', DATA_BLOCK_ENCODING => 'NONE', BLOOMFILTER => 'ROWCOL', REPLICATION_SCOPE => '0 true
  ', VERSIONS => '1', COMPRESSION => 'SNAPPY', MIN_VERSIONS => '0', TTL => 'FOREVER', KEEP_DELETED_CELLS => 'f
  alse', BLOCKSIZE => '16384', IN_MEMORY => 'false', BLOCKCACHE => 'true'}
1 row(s) in 0.0730 seconds
```

Note that there are two columns in the output DESCRIPTION and ENABLED. The value true displayed on the console is for the ENABLED column which informs the user whether the table is enabled or not. To see whether a table is defined in the cluster, use list command which lists all the tables in the HBase cluster and the following is a sample output.

```
hbase(main):018:0> list
TABLE
TestTable
TestTable1
TestTableRpl3
3 row(s) in 0.0600 seconds
=> ["TestTable", "TestTable1", "TestTableRpl3"]
```

To drop a table, the table need to be disabled first using the disable command before dropping using the drop command. If drop is attempted before the disable the shell will prompt with the message that the table is enabled.

```
hbase(main):019:0> drop 'TestTable'
ERROR: Table TestTable1 is enabled. Disable it first.'
 
hbase(main):020:0> disable 'TestTable'
0 row(s) in 1.3510 seconds
 hbase(main):021:0> drop 'TestTable'
0 row(s) in 0.2470 seconds
To alter a table, use alter command. The alter command can be used to make changes at table level and at the columnfamily level and it can also operate on multiple columnfamilies at the same time. Use the help 'alter' command to get more details. The following is an example of altering the number of versions that need to be stored for a particular columnfamily in a table.
alter table
hbase(main):025:0> alter 'TestTable',{NAME => 'cf', VERSIONS => 3}
Updating all regions with the new schema...
0/1 regions updated.
1/1 regions updated.
Done.
0 row(s) in 2.2800 seconds
```

Note that changes to some of the attributes require other actions. For e.g. changing the BLOCKSIZE will require a major compaction of the table if the change need to take effect immediately. Other changes like modifying the REGION_REPLICATION property requires the table to be disabled before altering the table and then enabling it
For basic data manipulations, the put, get, scan, delete commands can be used. The following puts two rows to a table, does a get a scan and a delete.

```
put get scan delete
hbase(main):036:0> put 'TestTable','r1','cf:c1','v1'
0 row(s) in 0.0300 seconds
hbase(main):037:0> put 'TestTable','r2','cf:c1','v2'
0 row(s) in 0.0060 seconds
hbase(main):038:0> get 'TestTable','r1'
COLUMN                                  	CELL
  cf:c1                                  	timestamp=1437492679670, value=v1
1 row(s) in 0.0340 seconds
hbase(main):039:0> scan 'TestTable'
ROW                                     	COLUMN+CELL
  r1                                     	column=cf:c1, timestamp=1437492679670, value=v1
  r2                                     	column=cf:c1, timestamp=1437492691520, value=v2
2 row(s) in 0.0620 seconds
hbase(main):041:0> delete 'TestTable','r1','cf:c1'
0 row(s) in 0.0630 seconds
```

During development if minor compaction need to be performed explicitly on table regions, the compact command can be used. Note: since compaction process will have performance impact use caution on when you invoke compaction in a production environment.
```
hbase(main):043:0> compact 'TestTable', 'cf'
0 row(s) in 3.5630 seconds
```

Also to improve data locality and performance major compaction of a table may have to be run and the following is an example. Again, major compaction is a resource intensive (CPU, IO, Memory and Network) process and should not be run in production without proper scheduling to minimize business impact.
```
hbase(main):044:0> major_compact 'TestTable'
0 row(s) in 1.1430 seconds
```

Balancing the region distribution of a table may help improving performance and it is a best practice to balance the regions before running major compaction explicitly to improve its performance. Be cautious on when you run balancer in production environment since it will impact performance.

```
hbase(main):045:0> balancer
true
0 row(s) in 59.1120 seconds
```

During development there may be a need to disable running of HBase balancer so that regions can be moved manually. Enabling and disabling of balancer can be accomplished using balance_switch command which takes in the value true|false as input.

```
hbase(main):048:0> balance_switch false
true
0 row(s) in 0.0300 seconds
```

Note that balance_switch command will return the previous value of whether the balancer is enabled or not. In the previous example the balancer was enabled and hence the return value of true.

To check the status of the HBase cluster, the status command can be used. The command can generate detailed, simple or summary status based on whether detailed|simple|summary parameter is passed.

```
hbase(main):052:0> status 'summary'
13 servers, 0 dead, 478.1538 average load
```
