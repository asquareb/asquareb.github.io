---
layout: post
title: "Challenges in concurrent programming II"
date: 2013-09-15 18:41:18 -0400
author: Biju Nair
comments: true
categories: [programming, software-design]
---

In addition to [race conditions](/blog/2013/08/15/challenges-in-concurrent-programming-i/), the second challenge in developing concurrent processing is deadlocks and they occur when the processes running concurrently need access to more than one resource. For e.g. if there are two resources R1 & R2 which will be accessed by two concurrent processes A & B,

- Lets assume process A acquired resource R1

- OS swaps processes A to start executing process B and processes B acquired resource R2
<!-- more -->
- Process B tries to acquire resource R1 which is help by process A. Process B gets blocked and gets swapped

- Process A gets scheduled and tries to acquire resource R2 and it will get blocked since R2 is held by process B

In this situation process A and B will not be able to proceed further and they are deadlocked. To generalize, the following four conditions need to be met for a deadlock situation to occur

- A resource can be held by only one process at a time and if not it should be available. This condition is called Mutual Exclusion

- A process can hold a resource and make a request to acquire other resources. This condition is called Hold and Wait Condition

- Resources held by a process can only be released voluntarily by the process and this condition is called no pre-emption condition

- A processes holding resources should request resources held by other processes forming a circular chain and this condition is referred to as circular wait

Solutions to overcome deadlocks can use one of the following strategies

- Including retry logic to acquire a resource if a process is not able to access it with in a certain time

- Rolling back processes in deadlock to a earlier point in execution and try again

- Plan allocation of resources so that deadlock can be prevented

- Programming such a way that one of the four conditions for deadlocks can be eliminated

In order to overcome the issues like race conditions and deadlocks in concurrent processing

- Thoroughly analyze the execution sequence of the programs and resource requirements using tools like state diagrams

- Identify the potential situations which can cause these issues

- Implement solutions using the best strategy which can prevent/resolve the issue
