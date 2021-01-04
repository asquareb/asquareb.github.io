---
layout: post
title: "Secure all applications please"
date: 2015-04-30 22:46:43 -0400
comments: true
author: Biju Nair
sharing: true
footer: true
categories: [security]
---
When you work with enterprises often you get to see batch applications storing credentials to login to systems like databases or messaging infrastructure or other enterprise applications in config files as plain text. Also these batch applications don’t get the same attention as customer facing applications when it comes to security. If you have similar application configurations and the thought is that these batch applications are behind the firewall in a DMZ and hence pose less risk, think again. As anyone who work in computer forensics/security can attest, most often data breach is perpetrated by an insider and these instances never get reported or get media attention. If you are looking for numbers here is a [summary of 2012 security incident report from Forrester](http://blogs.forrester.com/heidi_shey/13-01-07-a_2012_security_incident_recap_by_the_numbers). 

To de-risk scenarios like these, the solution doesn't have to be too complex. It can be a matter of following a simple process similar to the following across the enterprise,

<!--more-->

- Generate an application specific key for the particular machine on which an application will be scheduled to run.
- Store the key in a location which is accessible only to the service id (headless user id) which will run the application.
- Encrypt the password of credentials which will be used by the application to authenticate with critical systems using the key generated.
- Store the encrypted password and the location of the key in the application configuration file.
- In the application code, when required decrypt the password using the key. Once the decrypted password is used for authentication, destroy the decrypted password value so that application process memory dump doesn't expose the decrypted value.
- And have the application owner or sys admin own the responsibility for key generation, storage and encryption of passwords for the application.

Some may say that this is too simplistic and they may be correct. But here is something to think about. Have you wondered the use of a lock when road side assistance comes and opens your car when you loose your key? Locks are not for the small percentage of the population who will always find a way to break it, it is there to act as a barrier to the temptations of the vast majority of us. So even if this approach is simplistic it still acts as a barrier in a situation where there is nothing in place. If there are other solutions which can be put in place that you are comfortable with, it is even better, but please secure all applications. It can save someones life savings, identity, medical records or other critical data and the someone can be your friend, family, neighbor or even yourself!! With so much at stake on data, any vulnearabilty to breach is not an option anymore.

Note: A simple utility to create a barrier is available [here](https://github.com/bijugs/kaaval) for Java based applications.
