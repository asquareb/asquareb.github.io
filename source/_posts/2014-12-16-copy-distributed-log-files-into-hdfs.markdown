---
layout: post
title: "Copy distributed log files into HDFS"
date: 2014-12-16 21:23:51 -0500
author: Biju Nair
comments: true
categories: [hbase, monitoring]
---
In a distributed cluster like HBase, logs are created and written on individual nodes participating in the cluster. If users want to look at the log entries say for debugging purposes they would need to logon to individual nodes. Apart from the complexity of going through the logs on the individual nodes, users need to be provided access to all the nodes. One of the ways to mitigate these concerns is to copy the log files into a centralized location. 
<!--more-->

Apache Flume can be used to copy log files from individual nodes into HDFS. The following is an example Flume configuration which can be used to start Flume agents to copy HBase region server log files into HDFS. Note that changes will be required to this sample configuration based on cluster set-up.

{% codeblock %}

#
# Flume channel definition
#
agent.channels = memoryChannel
agent.channels.memoryChannel.type = memory
#
# Flume source definition
#
agent.sources = pstream
agent.sources.pstream.channels = memoryChannel
agent.sources.pstream.type = exec
agent.sources.pstream.command = tail -f /var/log/hbase/hbase-hbase-regionserver-$hostname.log
#
# Flume sink definition
#
agent.sinks = hdfsSink
agent.sinks.hdfsSink.type = hdfs
agent.sinks.hdfsSink.channel = memoryChannel
agent.sinks.hdfsSink.hdfs.useLocalTimeStamp = true
agent.sinks.hdfsSink.hdfs.path = hdfs://hdfs-service-name/user/flume/%y-%m-%d
agent.sinks.hdfsSink.hdfs.filePrefix = hbase-$hostname
agent.sinks.hdfsSink.hdfs.fileType = DataStream
agent.sinks.hdfsSink.hdfs.writeFormat = Text
agent.sinks.hdfsSink.hdfs.rollInterval = 86400
agent.sinks.hdfsSink.hdfs.rollSize = 0
agent.sinks.hdfsSink.hdfs.rollCount = 0

{% endcodeblock %} 

A daily log directory with the name `` yy-mm-dd `` will be created under `` /user/flume `` directory in HDFS. Log data from each individual nodes where Flume agent is run will be copied into individual files under the daily directory. By providing read access to the centralized HDFS directory in this case `` /user/flume ``, users will be able to go through the logs from all the nodes. Since each days logs are stored in seperate directories, users will be able to navigate easily. 

Inorder the Flume agent to copy the log data successfully, HDFS client components including ``hdfs-site.xml`` need to be installed on the individual nodes apart from Flume itself. In the case of HBase, HDFS components should be already available on the nodes since it is a pre-requisite.
