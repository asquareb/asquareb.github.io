---
layout: post
title: "In the Cloud?"
date: 2010-10-27 07:48:53 -0500
author: Biju Nair
comments: true
categories: [infrastructure-mgmt]
---
Every one is talking about Cloud and everyone wants to be on it!!!. At the same time when interacting with customers, there are questions raised which highlights the various levels of awareness and understanding of what Cloud delivery model means.

Some of the queries go like these
"Our systems are hosted by a vendor and we get the required capacity when required. Does this mean we are on the Cloud?"
<!--more-->
"We are using a web based product hosted by the vendor. So are we on Cloud?"

"We have virtualized our servers which gives the option of being able to switch on and off servers when required. Are we using a Cloud delivery model?"

"What is the general pattern seen in the industry to determine the type of workloads to be moved into private or public cloud?"

In order to help with understanding whether an organization is using Cloud delivery model, lets look at the definition of Cloud Computing and its attributes National Institute of Standards and Technology (NIST) provides a detailed definition of [Cloud computing](http://csrc.nist.gov/groups/SNS/cloud-computing/). But the one I like due to its simplicity is from CISCO which defines Cloud Computing as
"IT resources and services that are abstracted from the underlying infrastructure and provided "on-demand" and "at scale" in a multi-tenant environment"

Lets look at the key attributes of the definition

"**abstracted from the underlying infrastructure**" means that the users need not be aware of the underlying infrastructure (hardware or software) when they request or use a IT resource or service. Some of the techniques currently used are Virtualization for hardware abstraction, creating services for applications.

"**on-demand**" means resources can be provisioned when needed, released when no longer requires and charged only for the period used. The provisioning and release of the resources should have minimal management effort and service provider interaction. For e.g. for a web based retail business the provisioning may need to happen within minutes to cater an increase in transaction volume during holiday season and release the resources when the peak period is over.

"**at-scale**" means the illusion of infinite resource availability in order to meet what ever demands are made. For e.g. the volume of holiday shopping can be very high due to some popular gift items or a promotion and resources should be available to handle the volume seamlessly.

"**multi-tenant environment**" means that the resources are provided to many customers from a single implementation saving significant costs. The customers can be various business units for an enterprise IT team or multiple enterprises for an IT service provider.

If your IT delivery model or that of your service provider's delivery model satisfies these attributes, then it can be considered as using a pure Cloud delivery model. But at the sametime not all organizations have the same need as specified in the definition of Cloud. For e.g. if the query is whether you are using Cloud Computing when using a product hosted by the vendor, partially "yes" since you are not aware of the underlying infrastructure or whether the service is provided from a multi-tenant environment. At the sametime you may not have a requirement to be able to provision resources with in minutes with out service provider interaction.
