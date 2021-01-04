---
layout: post
title: "End-to-end arguments in system design"
date: 2016-02-07 18:41:26 -0500
author: Biju Nair
comments: true
sharing: true
footer: true
categories: [paper, system design, distributed systems]
---
[This paper](http://web.mit.edu/Saltzer/www/publications/endtoend/endtoend.pdf) presents the design principle regarding placement of functions in computer system design called "end-to-end argument".  The argument of the principle is that any application functions implemented at lower levels of a system may be redundant or of little value when compared with the cost of providing them at lower level.  The paper articulates the argument through requirements and examples in distributed systems like reliable data transmission, encryption, duplicate message detection, message sequencing, detecting host crashes, delivery receipts etc.
<!--more-->
Reliable Data Transmission

- Even if the communication network provides aid in coping with issues in data transmission like data buffering issues, processor or memory issues, loss of packet, host crashes through duplicate copies, timeout and retry, redundancy for error detection, crash recovery etc, at the end the data transfer application need to perform the check of the data transferred to claim it to be successful
- This makes the functions in the network layer for reliable data transfer redundant. More over these functions will impact other applications which will be using the network layer but doesn't require all the functions

Lower layers can implement function which can improve the performance of the application using it. For e.g. in the case of reliable data transmission

- The application need to make sure that the data transferred matches the source for e.g. by using checksums. If the checksum at the source and target don't match the data transfer need to be redone
- If functions can be included which will cost less but enhances the end to end reliability this will reduce the retries required to have the data data transferred reliably and hence improves the performance

Decisions to include functions in lower layers

- Need to take into account the cost of implementing it at the lower layer and the impact on other applications which may be using it
- Need to be made with the understanding that the higher level layers will have much more information than the lower level layers to guarantee a feature/functionality

In order to make these decisions i.e. whether to include functions in lower layers or let the application handle it at the end, application requirements of what need to be accomplished need to be well understood.
