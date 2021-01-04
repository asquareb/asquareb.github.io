---
layout: post
title: "Concurrency and why application developers need to know about it"
date: 2013-06-15 18:46:49 -0400
author: Biju Nair
comments: true
categories: [programming, software-design]
---
"Concurrent" in simple terms is defined as more than one thing happening or done at the same time. Working on a document and at the same time communicating with a colleague through a messaging system is an example of concurrency. But how does it relate to computers and programming? In its simple model, computers have interfaces like key board, mouse, touch screen, display unit for users to interact with, network interface to connect with other computers, storage to store and retrieve data and the hardware which includes the CPU to do processing. In order to manage the interactions with the various components, computers need to do multiple things at the same time. All the simultaneous interactions with devices and the users of computers are managed by operating systems. As any OS or device driver or system call developer can attest, concurrency is a challenge they face quite often.
<!-- more -->
Even though so much work has been done on concurrency in computer science, it is mostly hidden or not used by application developers since there were not many opportunities or challenges. Recent developments in the industry changes the status quo which requires all programmers to have a good understanding of concurrency in computers.

- Multi-core/multi-processor computers are the norm than the exception few years ago. What that means is application programmers need to think about leveraging and utilizing the multiple processors efficiently and in turn help complete tasks much faster.

- Due to more automation of business processes, ubiquitous use of electronic devices and new fields like wireless sensor networks, high frequency trading etc large volume of data is getting generated. There is a need to process the increased data volume much faster before it gets out dated.

- Related to data is the growing need for analytics to make sense of the large volume of data collected. Decisions are made more and more based on what is reflected in actual data i.e. relevant knowledge needs to be generated from the data at a much faster rate to be agile in decision making.

- Reduction in reliable hardware cost, virtualization and cloud computing paradigms is opening up opportunities to do at the application level which were not thought of a few years ago. Who would have ever thought a search engine like Google with millions of users can run on standard off the shelf hardware components a little more than a decade ago.

All these require application programmers to develop systems which can perform tasks concurrently so that the tasks can be completed quickly using the available resources efficiently. Since so much work has been done on concurrency and are being used in the areas like OS, it is a matter of learning from previous work and not re-invent them.  This will help leverage and ride the changes.
