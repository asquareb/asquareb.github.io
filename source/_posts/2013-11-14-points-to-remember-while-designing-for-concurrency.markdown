---
layout: post
title: "Points to remember while designing for concurrency"
date: 2013-11-14 17:52:27 -0400
author: Biju Nair
comments: true
categories: [programming, software-design]
---
Following are some of the points to consider while designing and developing solutions involving concurrent processing.

- Identify independent computations using techniques like task partitioning and data partitioning.
<!-- more -->
- Implement concurrency at the highest level possible.

- Don't make any assumptions on the number of cores available i.e the solution should be able to scale with the number of cores.

- Use the best processing model; multiple processes or multiple threads to solve the issue.

- If using multiple processes is found to be well suited, associate individual locks to variables which are shared.

- If threads are best suited for the solution, identify thread safe libraries that can be used. Also use thread local storage when ever possible.

- Never assume a particular execution order of the code which is one of the criteria for concurrent processing.

