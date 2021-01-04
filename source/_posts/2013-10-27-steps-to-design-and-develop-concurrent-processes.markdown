---
layout: post
title: "Steps to design and develop concurrent processes"
date: 2013-10-27 18:07:21 -0400
author: Biju Nair
comments: true
categories: [programming, software-design]
---
High level sequence of steps which can be followed to design/develop concurrent processes

- Start with a well tuned and fully functional serial processing design/code to solve the problem.
<!-- more -->
- Come-up with a design to solve the problem using concurrency.

- Calculate the speedup and efficiency of the design for concurrency.

- If sufficient speedup can be expected from the calculation, develop the code to do concurrent processing.

- Test the code and verify the results for various scenarios. Care need to be taken to verify situations like  correctness for loop executions, rounding errors of numerical  values, etc.

- Tune the code for performance and to minimize hotspots by identifying it through tools like profilers.

- Compare the performance and elapsed time of the serial code against the code for concurrency.

- If the performance has significant improvement implement concurrent processing instead of the serial processing code.

All these steps may not be applicable for a given situation, but following through them provides a good guideline.
