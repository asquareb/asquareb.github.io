---
layout: post
title: "Cassandra  - A Decentralized Structured Storage System"
date: 2013-01-23 21:44:21 -0800
comments: true
sharing: true
footer: true
categories: [databases, distributed systems, paper] 
---

##Cassandra had the following as the basic set of requirements##

  - Can run on commodity hardware and handle very high write throughput
  - Can scale with increase in the number of users
  - Should be able to replicate across geographical distributed data centers for fail over
<!--more-->

##Cassandra Data Model##

  - Table in Cassandra is a distributed multidimensional map indexed by keys
  - Row key is a string typically 16 to 36 bytes long
  - Columns which store values are grouped into column families of type simple or super column family
      - Super column families are column families within another column family
  - Columns can be sorted by applications either by name or timestamp
      - CRUD operations on Cassandra data can be performed by APIs
          - insert(table, key, rowMutation)
          - get(table, key, column)
          - delete(table, key column)
 
##System Architecture##

   - **Data Partitioning:** To scale incrementally, Cassandra partitions data using order preserving consistent hash function and distributes to the nodes in the cluster. The output range of the hash function is treated as a fixed circular space or “ring” (i.e. the largest hash value wraps around to the smallest hash value). Each node in the system is assigned a random value within this space which represents its position on the ring. Each data item identified by a key is assigned to a node by hashing the data item’s key to yield its position on the ring, and then walking the ring clockwise to find the first node with a position larger than the item’s position. This node is deemed the coordinator for this key. The application specifies this key and the Cassandra uses it to route requests. Thus, each node becomes responsible for the region in the ring between it and its predecessor node on the ring. The principal advantage of consistent hashing is that departure or arrival of a node only affects its immediate neighbors and other nodes remain unaffected. 
   - **Data Replication:** Data is replicated to N hosts which is preconfigured for availability and durability. The coordinator node for each key is responsible for replication to additional N-1 nodes. Cassandra provides various replication policies like Rack aware/unaware, data center aware and replicas are made according to the policy selected. In order to coordinate the ranges of keys for which each node is responsible for its replica, Cassandra system elects a leader amongst its nodes using ZooKeeper and it tells all nodes the ranges of keys it is responsible for. The metadata about the ranges each node is responsible for is stored locally and in ZooKeeper for fault tolerance. 
   - **Failure Detection:** In order to identify failures and avoid attempts to communicate with unreliable nodes Cassandra uses “The Φ accrual failure detector”. It calculates a threshold Φ based on the network and load conditions of the nodes monitored through the gossip messages send across nodes in the Cassandra cluster. Each node maintains a sliding window for this threshold to be calculated and depending on the desired threshold nodes can be identified as faulty and any further communication will be avoided.
   - **Node Bootstrapping:** When a new node is added to the cluster, it will use information about small set of nodes stored in ZooKeeper to know all the nodes and its position in the cluster. When a node starts for the first time, it chooses a random token for its position in the ring. For fault tolerance, the mapping is persisted to disk locally and also in Zookeeper. The token information is then gossiped around the cluster. This is how it knows about all nodes and their respective positions in the ring. This enables any node to route a request for a key to the correct node in the cluster. A new node added gets assigned a token such that it can alleviate a heavily loaded node. This results in the new node splitting a range that some other node was previously responsible for. 
   - **Data Persistence:** During writes data is written to commit logs in a sequential fashion for recovery before updating it to a in memory data structure. A dedicated disk is allocated on each node for commit log to maximize throughput. When the in memory data structure crosses a threshold, it is flushed to disk which also generates an index file for efficient lookup based on row key. Over time these files are merged by a background process to reduce the number of files that need to be accessed during reads. During reads the in memory data structure which will have the latest data will be read before the files on disk are accessed. In order to reduce the lookup of files which doesn’t store the data for the requested key, bloom filters are stored in each file and also in memory. 

   While cluster membership and failure detection module is implemented on top of a network layer which uses non-blocking I/O and the remaining modules rely on an event driven substrate which follows a SEDA architecture. All the system control messages rely on UDP and application related messages for replication and request routing relies on TCP. When a read/write request arrives at any node in the cluster the state machine morphs through the following states 

- Identify the node(s) that own the data for the key 
- Route the requests to the nodes and wait on the responses to arrive 
- If the replies do not arrive within a configured timeout value fail the re- quest and return to the client 
- Figure out the latest response based on timestamp 
- Schedule a repair of the data at any replica if they do not have the latest piece of data. 
  
##References##

- [Cassandra - A Decentralized Structured Storage System](https://www.cs.cornell.edu/projects/ladis2009/papers/lakshman-ladis2009.pdf)
