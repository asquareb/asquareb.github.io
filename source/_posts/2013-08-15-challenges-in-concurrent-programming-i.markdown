---
layout: post
title: "Challenges in concurrent programming - I"
date: 2013-08-15 18:38:19 -0400
author: Biju Nair
comments: true
categories: [programming, software-design]
---

Inherently concurrent processing will involve shared resources like memory and will lead to race conditions. A simple example would be an accumulator variable which needs to be read by all the processes running concurrently, adding some value to it and storing back the result. The race condition occurs when
<!-- more -->
- Process A reads the accumulator value

- OS stops process A and schedules process B to run which also reads the accumulator value

- Now OS switches process A to run which adds some value to the accumulator value and stores back

- By the time when Process B gets scheduled to run, it will not see the updated accumulator value resulting in incorrect value stored back by process B

Such race conditions will break one of the criteria concurrent processes need to satisfy namely, any order of execution of the processes participating in concurrent processing should generate the same result. To avoid race conditions processes need to be synchronized and any solution to prevent race condition need to satisfy the following four conditions.

Lets define the code segment executed by each process to access any shared resource as the "critical section" of the code

- Only one process should be allowed to execute its critical section of the code at any time

- There should be no assumptions made regarding the number of CPUs, timing of executions of the processes or the order

- No process should be prevented from executing its critical section of the code

- No process which is executing code in its non critical section should be able to block execution of another process

Some of the code constructs, techniques which can used to synchronize process executions are semaphores, mutex (a type of semaphore), monitors,  barriers and producer-consumer which uses messaging. Based on the programming language you choose one or more of these constructs may already be available or can be developed.
