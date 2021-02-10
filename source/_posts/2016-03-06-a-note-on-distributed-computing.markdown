---
layout: post
title: "Note on Distributed Computing"
date: 2016-03-06 16:18:22 -0500
author: Biju Nair
comments: true
sharing: true
footer: true
categories: [paper,distributed sytems]
---
A distributed system is a collection of independent computers that appears to its users as a single coherent system. [This paper](http://theory.stanford.edu/people/jcm/cs358-96/spring-os.ps) argues that the objects in a distributed object oriented system form a single ontological class where all entities can be described by the specification of the set of interfaces of the objects and the semantics of operation is mistaken. This vision of unified objects for distributed systems is centered around the principles that
<!-- more -->
  - There is a single natural object oriented design for a given application, irrespective of context in which the application will be deployed
  - Failure and performance issues are tied to the implementation of the components of the application and can be left out of initial design
  - The interface of an object is independent of the context in which the object is used

While this view is a natural extension of object oriented design, it doesn't take into account the important issues in distributed systems namely

  - Latency
    - Ignoring the difference between the performance of local and remote invocations can lead to designs whose implementations will have performance problems because of large amount of communication between components that are in different address spaces and on different machines.
  - Memory access
    - Disparate address spaces both local and remote makes accessing memory locations/objects a bigger challenge
    - This would require a common component like DSM which would translate memory pointer/object locations
    - The other option is that the programmer is aware of the local and remote access which breaks transparency
  - Partial failure and concurrency due to lack of common agent
    - Interfaces of components can be designed as if all are local resulting in unhandled catastrophic failures
    - Or design the interfaces as if all objects are remote resulting in the undesirable overhead on components which are local
    - Partial failure also brings up the challenge of bringing back to an acceptable state which is not there in non distributed systems

Historically there is a desire to merge programming and computational models of local and remote computing. For e.g. communications protocol development has followed

  - A path where integration with the current programming language is emphasized
  - The other path where solving the inherent problems in distributed computing is emphasized

These two approaches can be used in distributed object oriented systems by accepting the fact that there are irreconcilable differences in local and distributed computing

  - Using IDLs which can be used to define local and remote objects and the IDL compilers which can vary its output based on object location
  - Objects which are local but on a different address space can be considered as a separate category of object by the IDL compilers
  - Developing tools like the ones which can identify the interaction between objects so that they can be designed to be local or remote
  - Programmers being aware of the location of the objects so that they can think about the problems differently
