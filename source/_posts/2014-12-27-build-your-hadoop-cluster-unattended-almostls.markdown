---
layout: post
title: "Build your Hadoop cluster unattended (almost!!)"
date: 2014-12-27 21:13:36 -0500
author: Biju Nair
comments: true
sharing: true
footer: true
categories: [Chef, Infrastructure-mgmt, Hadoop]
---
If you are a developer who would like to have a Hadoop cluster or a dev lead who would like everyone in your team a cluster of their own without going through the hassle of creating machines, networking them, install the required software components, [chef-bach](https://github.com/bloomberg/chef-bach) is an option you want to try.
<!-- more -->
Chef-Bach is a set of [Chef](https://www.chef.io/) cookbooks which can be used bring up a Hadoop cluster. It is open source software and that means it can be customized to your needs. By default, a "Test-Laptop" configuration will bring up a three node Hadoop cluster based on [HDP 2.0](http://hortonworks.com/products/releases/hdp-2-0-ga/) on a laptop or machine with enough resources like 16 GB RAM and quad core processor. Behind the scenes, chef-bach will create VM machines using VirtualBox, configures the network, PXE boots the cluster nodes, installs OS and deploys the Hadoop components with out any user intervention (almost). 

The total time to create a "Test-Laptop" cluster is around 3 hours and that means at anytime the cluster can be destroyed and rebuilt without much loss in time or productivity of developers. If this is something of interest to you, refer to the instructions to perform the cluster creation process [here](https://gist.github.com/cbaenziger/816e7c42913433f5743f). Also users will be able to create a cluster with [more than 3 nodes](https://gist.github.com/bijugs/27f44d51b69402eb90ab) or [add new nodes](https://gist.github.com/bijugs/27f44d51b69402eb90ab) to the cluster after the cluster has been created. 

Even though test cluster creation is mentioned as the primary use case so far, nothing prevents someone to use chef-bach to create production clusters and for that matter it is currently being used in production environment. Some of the notable features which makes it production capable include 

- deployment of key components like mysql server, graphite, zabbix etc in HA mode 
- monitoring and triggering using Zabbix and Graphite 
- all JMX and server statistics made available on Graphite so that graphs using various data points can be created for better insight
- rolling restart of various Hadoop components to prevent unavailability during configuration changes
- being able to run chef as daemon process at regular intervals so that all the nodes are kept updated
- copy of logs from various nodes into a centralized location (HDFS) so that users can access them for debugging
- option to create a [Kafka](https://kafka.apache.org/) cluster

As a open source project you can also get involved and influence the future direction of this project.  

More notes on this category can be found [here](http://blog.asquareb.com/blog/categories/chef/).
