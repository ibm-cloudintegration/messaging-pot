---
title: IBM MQ Appliance
toc: false
sidebar: labs_sidebar
folder: pots/mq-appliance
permalink: /mq_appl_pot_overview.html
summary: Overview of IBM MQ Appliance PoT
---

# IBM MQ Appliance Proof of Technology Overview

This guide and associated VMware images help you get started with IBM® MQ Appliance V9.1.2.  

There are three environments to choose from: 

* Start at appliance power-up and configure the appliance 

	[Setup the IBM MQ Appliance](mq_appl_pot_lab1.html)

* Appliances are setup and you want to configure them for high availability 

	[Configure High Availability for the IBM MQ Appliance](mq_appl_pot_lab2.html)

* Appliances are setup and configured for high availability, and now you want to set up a disaster recovery environment 

	[Configure Disaster Recovery for the IBM MQ Appliance](mq_appl_pot_lab8.html)

# Introduction

The IBM MQ Appliance provides a simplified messaging solution by
combining many of the benefits of IBM MQ with those of a physical
appliance. Like IBM MQ, the IBM MQ Appliance provides a rapid, reliable,
security-rich infrastructure, but unlike IBM MQ, it combines software
with hardware. Deployable in less than 30 minutes, the IBM MQ Appliance
saves customers from having to build their own servers, or needing
extensive local MQ expertise when they deploy to partners or remote
locations.

The IBM MQ Appliance has the following benefits:

*  Seamless integration: integrates with existing MQ networks and
    clusters

*   Simple, out-of-the-box High Availability: with paired connectivity
    to another appliance, delivering personalized role-based monitoring
    and configuration

*   Simple, out-of-the-box Disaster Recovery: with flexible topologies
    and use cases making adding DR easy

*   MQ Console: a browser-based user interface, offering personalized
    role-based monitoring and configuration

*   Simpler maintenance: fix packs delivered as certified firmware
    updates onto a locked-down appliance

*   Maximum performance out-of-the-box: pre-optimized with no tuning or
    additional configuration needed

Customers can choose between two appliances, according to the needs of
their business:

*   IBM MQ Appliance M2002A: A high-end solution for enterprise use

*   IBM MQ Appliance M2002B: Lower-end solution for branch office or
    factory deployment

### Audience

This session is specifically designed for IT Professionals such as
system architects, system designers and managers who are looking for a
solution to satisfy their enterprise messaging and data movement
requirements. No detailed knowledge of IBM software is required.
However, it is recommended that participants have an understanding of
their business needs concerning messaging and data movement. A basic
understanding of IBM MQ is also assumed.

### Icons 

The following symbols appear in this document at places where additional
guidance is available.

  
  Keep an eye out for the tooltips below. These will help guide you through common mistakes, provide tips and helpful context to the task at hand. 

{% include note.html content="This is a note.
" %}

{% include important.html content="This is important. This symbol calls attention to a particular step or command. For example, it might alert you to type a command carefully because it is case sensitive." %}

{% include warning.html content="This is a warning -- something specific to pay attention to." %}

{% include troubleshooting.html content="This is trouble.
" %}

### IP Addresses

This attachment details all of the IP addresses you need for the
appliances in the labs.

Use the tables below to guide you through the initial set up of MQAppl1
in Lab 1 and also MQAppl2 and MQAppl3 if you choose to manually
configure the appliances for the HA and DR labs.

You do not have to configure MQAppl2 and MQAppl3 unless you choose to
continue using the initial environment for the HA and DR labs. A
separate environment has been created for you to use if you choose not
to.

The network adapters are described here. Pay particular attention to
eth1, eth2, and eth3 and eth4. They are the adapters used for HA and DR.

Ensure you enter the addresses exactly as shown here in conjunction with
the lab instructions.

### MQAppl1 Addresses

Note that the IP addresses of eth1 through eth4 MUST be entered in CIDR
notation as shown below. 

|Virtual Adapter Network Name | Appliance Ethernet Interface | Usage | DHCP | IP Address | Gateway Addresses |
|:---------------------------:|:----------------------------:|:-----:|:----:|:----------:|:-----------------:|
| Network 1 | eth0 | Management and client traffic | Yes | 10.0.0.1 | |
| HA1 | eth1 | HA primary connection | No | 10.0.1.1 | 10.0.1.254 |
| HA2 | eth2 | HA alternate connection | No | 10.0.2.1 | 10.0.2.254 |
| HArep | eth3 | HA replication connection | No | 10.0.3.1 | 10.0.3.254 |
| DRrep | eth4 | DR replication connection | No| 10.0.4.1 | 10.0.4.254 |

### MQAppl2 Addresses

Note that the IP addresses of eth1 through eth4 MUST be entered in CIDR
notation as shown below. 

|Virtual Adapter Network Name | Appliance Ethernet Interface | Usage | DHCP | IP Address | Gateway Addresses |
|:---------------------------:|:----------------------------:|:-----:|:----:|:----------:|:-----------------:|
| Network 1 | eth0 | Management and client traffic | Yes | 10.0.0.2 |  |
| HA1 | eth1 | HA primary connection | No | 10.0.1.2 | 10.0.1.254 |
| HA2 | eth2 | HA alternate connection | No | 10.0.2.2 | 10.0.2.254 |
| HArep | eth3 | HA replication connection | No | 10.0.3.2 | 10.0.3.254 |
| DRrep | eth4 | DR replication connection | No| 10.0.4.2 | 10.0.4.254 |

### MQAppl3 Addresses

Note that the IP addresses of eth1 through eth4 MUST be entered in CIDR
notation as shown below. 

|Virtual Adapter Network Name | Appliance Ethernet Interface | Usage | DHCP | IP Address | Gateway Addresses |
|:---------------------------:|:----------------------------:|:-----:|:----:|:----------:|:-----------------:|
| Network 1 | eth0 | Management and client traffic | Yes | 10.0.0.3 | |
| HA1 | eth1 | HA primary connection | No | 10.0.1.3 | 10.0.1.254 |
| HA2 | eth2 | HA alternate connection | No | 10.0.2.3 | 10.0.2.254 |
| HArep | eth3 | HA replication connection | No | 10.0.3.3 | 10.0.3.254 |
| DRrep | eth4 | DR replication connection | No| 10.0.4.3 | 10.0.4.254 |

[Return to MQ Appliance Menu](mq_appl_pot_overview.html)

  


