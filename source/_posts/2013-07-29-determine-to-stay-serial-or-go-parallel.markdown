---
layout: post
title: "Determine to stay serial or go parallel"
date: 2013-07-29 18:12:46 -0400
author: Biju Nair
comments: true
categories: [programming, software-design]
---
Given that concurrent programming involves unique challenges like race conditions and deadlocks, it would be advantageous to determine whether concurrent processing of a problem provides better performance than serial processing. Speedup is the metric which can be used to make this comparison. This involves identifying the best solutions to solve the problem through serial processing and concurrent processing.
<!-- more -->
Speed up in simple terms is the ratio of serial execution time to parallel execution time and is expressed as a multiplier. For e.g if it takes 1000 seconds and 100 seconds to complete a problem through serial and concurrent execution then the speedup is 10 times normally indicated as 10x. In order to determine an approximate speedup without actually developing and executing any programs, two laws Amdhal's and/or Gustafson's can be used. It would require a good understanding of the percentage of time the concurrent processing solution can execute concurrently. Even though  the values derived out of these laws may not be perfect with out actual execution, it would give a good idea as to whether one can pursue developing the concurrent processing solution.

<h2> Amdhal's Law: </h2>

Amdhal's law states that Speedup <= 1/((1-PCTpar)+(PCTpar/P))

- where PCTpar is the percentage of code which will run concurrently

- P is the number of processors/cores

For e.g. if the percentage of code which will run concurrently is 90 and there are 8 cores available on the machine the speedup <= 1/((1-0.9)+(0.9/8) which is 4.7x speedup compared to serial processing.

<h2> Gustafson's Law: </h2>

- Gustafson's law covers some of the criticisms about Amdhal's law like not taking into account the increase in the data processed with the increase in the number of cores.

- It states that Speedup <= P + (1-P) S

- where P is the number of processors/cores as in Amdhal's law

- S is the percentage of time spend in serial execution

For e.g. if the percentage of time which will be spent in serial execution is 1% and if there are 32 cores then the speedup will be <= 32+(1-32)*0.01 which is 31.69x speedup

- Another metric which is can be used to measure concurrent processing is its efficiency which provides the percentage utilization of the computational resources.

- Efficiency is defined as Speedup/number of cores and is expressed as a percentage. For e.g. the efficiency of 31.69x speedup on 32 core is 99% efficient.

By doing this calculations upfront one can determine whether to go concurrent or stay serial.
