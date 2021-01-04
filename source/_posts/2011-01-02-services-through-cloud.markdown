---
layout: post
title: "Services through cloud"
date: 2011-01-02 08:16:44 -0500
author: Biju Nair
comments: true
categories: [infrastructure-mgmt]
---
Even though there are various types of Cloud service vendors available in the market, they all can be mapped back to the following three basic Cloud service models.

<!--more-->
**Cloud Software as a Service (SaaS)**: The capability provided to the consumer is to use the service provider’s applications running on a cloud infrastructure. The applications are accessible from various client devices through a thin client interface such as a web browser. The consumer does not manage or control the underlying cloud infrastructure including network, servers, operating systems, storage, or even individual application capabilities, with the possible exception of limited user-specific application configuration changes. Some of the examples are Google mail, ADP Workforce Management System, Salesforce.com.
SaaS solutions are well suited for applications using no industry specific logic like a Email system or which follows industry standard workflow with minimal customization like a time or expense tracking system.

**Cloud Platform as a Service (PaaS)**: The capability provided to the consumer is to deploy onto the cloud infrastructure consumer-created or acquired applications created using programming languages and tools supported by the service provider. The consumer does not manage or control the underlying cloud infrastructure including network, servers, operating systems, or storage, but has control over the deployed applications and possibly application hosting environment configurations. Google App Engine is an example of PaaS where consumers can develop and deploy web applications using the supported languages like Java and Python and the APIs provided by the App engine.
Platforms made available by vendors will have limitations and also the applications developed on these platforms may end up using proprietary vendor APIs which makes moving the applications to another platform difficult. If a suitable platform is found for an application, using industry standards to develop the same will mitigate vendor lock-in concerns.

**Cloud Infrastructure as a Service (IaaS)**: The capability provided to the consumer is to provision processing, storage, networks, and other fundamental computing resources where the consumer is able to deploy and run arbitrary software, which can include operating systems and applications. The consumer does not manage or control the underlying cloud infrastructure but has control over operating systems, storage, deployed applications, and possibly limited control of select networking components (e.g., host firewalls). Amazon EC2 is an example of IaaS.
There can be situations where products required to develop an application like DBMS, Application Servers, Workflow Engines may not have a Cloud based licensing. This needs to be looked into so that the full cost benefit of using Cloud based infrastructure can be realized.
