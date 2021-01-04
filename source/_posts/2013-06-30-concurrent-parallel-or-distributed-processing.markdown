---
layout: post
title: "Concurrent Parallel or Distributed Processing"
date: 2013-06-30 18:52:25 -0400
comments: true
categories: [programming]
---

Reading through literature and books on concurrency and programming there are instances where terminologies are used interchangeably which sometimes cause confusion. Here are some of the terms and how I understand/use them
<!-- more -->
<h2>Concurrent Processing:</h2> 
Execution of two or more processes concurrently to achieve a single goal is concurrent processing. But if processes are running on a computer with a single processing unit which is required to perform any processing how can multiple processes get executed at the same time. That requires the definition of "concurrent" to be relaxed so that one or more things appear to be happening or done at the same time. A good analogy would be when we say we worked on a document while attending a conference call concurrently. At any point in time we can do only one task at any instant but we can still make progress on both tasks based on the opportunities available like not having to listen some part of the conference call which appears like work being done concurrently. Such opportunities are also available when multiple processes are executed in a system like some processes waiting for user action or waiting for a network response or waiting for a response from storage devices during which time other processes can be executed by the processors and all the processes appears to be running concurrently. Using programming techniques and creating programs which can run concurrently is concurrent programming.
<h2>Parallel Processing:</h2> 
Execution of two or more processes in parallel to achieve a single goal is parallel processing. Since by definition the processes need to be executed in parallel, parallel processing requires more than one processor. A simple analogy will be a group of engineers working in parallel to solve a problem. Parallel processing is a type of concurrent processing with the requirement that it needs more than one processor so that processes can be executed in parallel at any given time. The multiple processors can be part of a single computer or can be from a network of computers.
<h2>Distributed Processing:</h2>
Execution of two or more processes in parallel on two or more computers  to achieve a single goal is distributed processing. The computers on which the processes are executed can have one or more processors. Teams of engineers working on various components of a system is a simple analogy in this case. Again like parallel processing, distributed processing is a type of concurrent processing.
If you are familiar with object oriented programming paradigm, concurrent processing can be considered as the base and parallel and distributed processing are derived from the type concurrent processing. It may be a bit of a stretch, but still an easy way to remember.
