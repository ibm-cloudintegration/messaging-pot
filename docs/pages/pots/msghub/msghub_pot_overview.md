---
title: IBM Event Streams
toc: false
sidebar: labs_sidebar
folder: pots/msghub
permalink: /msghub_pot_overview.html
summary: Introduction to IBM Event Streams
applies_to: [developer,administrator]
---

![](./images/pots/msghub/msghub-overview.png)

## Introduction

This Proof of Technology (PoT) provides a hands-on experience for those needing to understand IBM Event Streams on IBM Cloud and/or IBM Cloud Private. 

There are two parts to the PoT:
 
* Part 1 - IBM Event Streams on IBM Cloud
	* Labs 1 and 2
* Part 2 - IBM Event Streams on IBM Cloud Private (ICP)
	* Labs 3 - 8

Each part requires a different demo environment in Bluedemos. If you are running this lab as part of a PoT event, the instructor will provide a URL(s) for you to use to access a Skytap environment. Open a browser and navigate to the URL which will open the Skytap environment. 

If you are running this lab independent of an event, click on the appropriate link below to reserve a demo which will provision a Skytap environment for you.

* [Reserve your Part 1 demo environment](https://bluedemos.com/show/1601/)
* [Reserve your Part 2 demo environment](https://bluedemos.com/show/1806/)

### Part 1 of the Proof of Technology (PoT)
Part 1 of the Proof of Technology (PoT) provides a hands-on experience for those needing to understand how to utilize IBM Event Streams on IBM Cloud and the associated MQ Bridge. Lab 1 must be completed in order to do Lab 2 (MQ Bridge).

In order to complete this PoT you will need to have access to the **Xubuntu 64-bit 14.04** virtual machine image that may be found in the Skytap Environment that your instructor has provided you.  You will also need an IBM Cloud account.  Complete the steps described in the [Prerequisites](msghub_pot_prereqs.html) document if an instructor has not provided an IBM Cloud account for you to use.

{% include note.html content="This lab is focused on creating and using an IBM Event Streams on IBM Cloud instance.  It does not focus on the installation and use of Apache Kafka.  Refer to the following URL for information on Apache Kafka: [https://kafka.apache.org/](https://kafka.apache.org/)" %}

### Part 2 of the Proof of Technology (PoT)
Part 2 of the Proof of Technology (PoT) provides a hands-on experience for those needing to understand how to utilize IBM Event Streams on IBM Cloud Private (ICP). The associated Skytap environment includes a pre-installed and configured ICP foundation upon which you will install IBM Event Streams. Part 2 starts with Lab 3 and all labs must be completed sequentially.


## How it Works

IBM Event Streams is a scalable, high-throughput message bus that is hosted on IBM Cloud. IBM Event Streams provides an open-source, high-throughput messaging system which delivers a low-latency platform for handling real-time data feeds.

IBM Event Streams provides the following features:

* Fast, scalable, fully managed messaging service, based on Apache Kafka

	Event Streams sits on Apache Kafka, an open-source, high-throughput messaging system which provides a low-latency platform for handling real-time data feeds.

* Event Streams provides 3 interfaces through which messages can be produced and consumed. 


	1. The native Kafka interface for Kafka clients
	2. An MQ Light API for MQ Light clients
	3. A REST API.

* Streaming analytics integration.	

	IBM Event Streams uses the power of the Apache Spark and Streaming Analytics services alongside Kafka to build high-performance scalable streaming analytics solutions.

* Connectivity with a wide range of services.

	IBM Event Streams will integrate with multiple services on the IBM Cloud platform.


[Review the prerequisites for Part 1 of PoT](msghub_pot_prereqs)

[Continue to Lab 1](msghub_pot_lab1.html)

[Continue to Lab 3](msghub_pot_lab3.html)