---
layout: post
title: "Scale in Distributed Systems"
date: 2016-01-10 18:51:35 -0500
author: Biju Nair
comments: true
sharing: true
footer: true
categories: [paper, distributed systems]
---
[This paper](http://clifford.neuman.name/papers/pdf/94--_scale-dist-sys-neuman-readings-dcs.pdf) looks at scale and how it affects distributed systems including highlights of how how scale is addressed in existing systems.
A system is said to be **scalable** if it can handle addition of resources and users without suffering noticeable loss in performance or increase in administrative complexity. Scale also affects the way users perceive the systems. For e.g. as the number of objects accessible grows it becomes increasingly difficult to locate the objects of interest.
<!--more-->
- Definitions
  - A **distributed system** is a collection of computers, connected by a computer network, working together to collectively implement some minimal set of services.
  - A service or resource is **replicated** when it has multiple logically identical instances appearing on different nodes in a system
  - A service is **distributed** when it is provided by multiple nodes each capable of handling a subset of the requests for service. A distribution function maps requests to the subset of the nodes that can handle it 
  - **Caching** is a temporary form of replication used to save and reuse of query results on nodes. Caching need to use validation techniques to make sure that the data saved are current
- Effects of Scale
  - Reliability
    - Systems should not cease to operate just because nodes are unavailable
    - Reliability can be improved by increasing the autonomy of the nodes and replication
  - System Load
    - System query load increases with increase in amount of data, nodes, services
    - Replication, distribution and caching can be used to reduce the number of requests that need to be handled by each node
  - Administration
    - With increase in number of nodes, administration of users, services and systems becomes complex
    - Complexity in administration can be reduced by maintaining common information centrally
  - Heterogeneity 
    - With scale nodes part of the system can not only of different hardware but can also run different OS and different versions of OS
    - **Coherence** an approach which expects nodes in a system support a common interface is one which is used to deal with heterogeneity
      - Common instruction set
      - Common execution abstraction 
      - Support a common set of protocols
- Distributed system components affected by scale
  - Naming and directory services
  - Authentication
  - Authorization
  - Accounting
  - Communication
  - Remote Resources

_Scaling of all the components can be improved by replication, distribution and caching_

Key points to remember while building scalable systems

- Replication
  - Replication important resources
  - Distribute the replicas
  - Use loose consistency
- Distribution
  - Distribute across multiple servers
  - Distribute evenly
  - Exploit locality
  - Avoid upper level of hierarchies
- Caching
  - Cache frequently accessed data
  - Consider access patterns when caching
    - amount of data accessed together, read to write ratio, likelihood of conflicts, number of simultaneous users
  - Cache timeouts
  - Caching at multiple levels
  - Look first locally
  - Data cached extensively must be changed less frequently
- Avoid global broadcast
- Shed load but not too much: perform computation where it suits better
- Support multiple access mechanisms
- Keep users in mind

Evaluating distributed systems

  - Use of the system
    - Growth of queries as the system grows
    - Central servers in the system and issues with replication
  - Data
    - Increase in data and how it increases data maintained in each node in the system
    - Increase in query time with increase in data size
    - Data update process and how it scales 
    - Cache data invalidation and query performance
  - Administration
    - Does the system require a centralized admin system?
    - Is it practical in the environment in which the system is used?
