---
layout: post
title: "JDBC Connection to Apache Phoenix"
date: 2016-03-31 12:27:36 -0400
comments: true
author: Biju Nair
sharing: true
footer: true
categories: [phoenix, hbase]
---
Phoenix provides a JDBC driver for Java client and hence can be connected to Phoenix by following the steps required to get a JDBC connection. As with JDBC drivers for other DBMS, there are are some Phoenix specific requirements to get a JDBC connection. For a non secure HBase cluster the Phoenix JDBC connection string should be of the form ``jdbc:phoenix:<ZK-QUORUM>:<ZK-PORT>:<ZK-HBASE-NODE>``. The following is the code snippet to get a Phoenix JDBC connection object for a non secure HBase cluster.
<!--more-->
```
Connection con = DriverManager.getConnection("jdbc:phoenix:nodea,nodeb,nodec:2181:/hbase");
```
To connect to a secure HBase cluster using a Kerberos user principal and keytab, the Phoenix JDBC connection string should be of the form ``jdbc:phoenix:<ZK-QUORUM>:<ZK-PORT>:<ZK-HBASE-NODE>:principal_name@REALM:/path/to/keytab``. The following is the code snippet to get a Phoenix JDBC connection object for a secure HBase cluster.
```
con = DriverManager.getConnection("jdbc:phoenix:node1,node2,node3:2181:/hbase:peace@REALM.COM:/home/peace/peace.keytab);
```
If Kerberos principal and keytab is not used to connect to a secured HBase cluster, then the user running the code to make the connection should be defined in the Kerberos KDC and should have a valid TGT. The user running the code can verify whether they are in the correct KDC and have a valid TGT by running ``klist`` command . One key item to note is that to access a secure HBase cluster, the hbase-site.xml and core-site.xml of the target HBase cluster should be available in the classpath of the application. 
