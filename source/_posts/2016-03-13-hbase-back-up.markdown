---
layout: post
title: "HBase Back-up"
date: 2015-10-24 11:02:17 -0400
comments: true
author: Biju Nair
sharing: true
footer: true
categories: hbase
---
Often during development and sometimes in production backup of a HBase table need to be made, for e.g., to run a test code against a table during development and being able to restore the original data if some thing went wrong or create a clone of the existing table or move table data to a new development cluster. HBase provides the option of taking snapshot of tables which can be used in such scenarios. The following are the various hbase shell commands to accomplish some of the common requirements.
<!--more-->

**Create a snapshot of HBase table**
{% codeblock %}
$ hbase shell
... 
hbase(main):001:0> snapshot 'tableName', 'table-snapshotname'
0 row(s) in 1.1060 seconds
{% endcodeblock %}
For easier identification it is a good practice to create snapshot with table name, creation date, creation time in the snapshot name.

**Restore data from snapshot**
{% codeblock %}
$ hbase shell
 
hbase(main):001:0> disable 'tableName'
0 row(s) in 1.1000 seconds
hbase(main):001:0> restore_snapshot 'table-snapshotname'
0 row(s) in 2.1060 seconds
hbase(main):001:0> enable 'tableName'
0 row(s) in 1.0000 seconds
{% endcodeblock %}

Note that the table need to be disabled to restore the table data from snapshot. Also note that any updates to the table data after the snapshot will be lost once the restoration is complete.

**Clone table from snapshot**
{% codeblock %}
$ hbase shell
... 
hbase(main):001:0> clone_snapshot 'table-snapshotname', 'newTableName'
0 row(s) in 2.1060 seconds
{% endcodeblock %}
A new table will be created with the attributes of the original table from which the snap shot was made and the data from the point in time of the snapshot will be restored.

**List all the available snapshots for a table**
If multiple snapshots were made on tables and would like to see the list of available snapshots
{% codeblock %}
$ hbase shell
 
hbase(main):001:0> list_snapshots
SNAPSHOT                                     TABLE + CREATION TIME
  emp-snapshot-073115                        emp (Fri Jul 31 16:07:14 -0400 2015)
 
1 row(s) in 2.1060 seconds
{% endcodeblock %}
