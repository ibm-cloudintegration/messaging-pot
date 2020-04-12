---
title: Replicated Data Queue Managers (RDQM) for Disaster Recovery 
toc: false
sidebar: labs_sidebar
folder: pots/mq-ha
permalink: /mq_ha_pot_lab3.html
summary: Replicated Data Queue Managers (RDQM) for Disaster Recovery
applies_to: administrator
---

# Introducing IBM MQ Replicated Data Queue Manager

## Introduction

This lab provides a demonstration of a new approach to Disaster Recovery in MQ on Linux, with the following key features:

* Use of Distributed Replicated Block Device (DRBD) storage rather than network shared storage* This is still using a Replicated Data Queue Manager (RDQM):	* Takeover will be manual, not automatic	* Both asynchronous and synchronous replication is supported	* An RDQM is active on only one node at any one time
	* Each node can run different active RDQMs	* An individual DR RDQM is created to use one style of replication and it cannot be changed without recreating the RDQM	* In 9.0.5 (through 9.1.2) an RDQM can be either HA or DR but not both	* As only two nodes are involved, it will be possible to get into a split-brain situation; but only if a user has chosen to promote a DR Secondary and start a DR RDQM when the DR network is disconnected and the DR RDQM is still running, or is also started where it was Primary.

The goals for RDQM-DR are:1. Allow an RDQM to be created which is configured to replicate its data to a single Secondary instance at a given IP address
	* Asynchronous replication is supported provided the latency is no more than 50ms for a round trip time
	
	* Synchronous replication is subject to the same 5ms limits on latency as it is for HA
2. Allow manual control of when a DR Secondary becomes a DR Primary and can then run the RDQM
In this lab, instructions are provided to show the setup for both.For this lab exercise, node 02 has had some of the ‘pre-configuration’ already performed.

### Summary of lab environment

1. 3 RHEL 7.4 x86_64 systems running in Skytap: 

	* miqmp  - This will be our primary node	* miqms  - This will be a secondary node

	(The third system is not used for this lab.)

1. VMWare Workstation virtual networks: 
	
	|Name   | Type     |  Subnet  | DHCP |
	|:-----:|:--------:|:--------:|:-----:|
	|VMnet3 |Host-only | 10.0.3.0 |no     |
	|VMnet4 |Host-only | 10.0.4.0 |no     |
	|VMnet8 | NAT      | 10.0.0.0 |no     |
	
1. Network interfaces:

	|Interface Purpose | Interface Name |  miqmp (Primary node)  | miqms (Secondary node) | 
	|:------:|:--------:|:--------:|:-----:|:--------|
	|Administration | ens34 | 10.0.0.1 |10.0.0.2 |
	|DR Replication | ens38 | 10.0.4.1 |10.0.4.2 |
	|MQ Fixed IP | ens37  | 10.0.3.1 |10.0.3.2 |

DR interfaces are used as follows:* DR Replication - for synchronous / asynchronous data replication (the higher the bandwidth the better and the lower the latency the better)

 {% include note.html content="Hosts miqmp, miqms are tied to the Administration IP addresses above" %}
### Pre-configuration steps 
The following steps are necessary for configuring RDQM, and are shown for your reference. They have **already been completed** on the VMs. 

* Although not required for this Lab, the following Pacemaker dependencies required for RDQM HA have already been installed. This list should be sufficient for a standard installation of RHEL 7.6 Server or Workstation. For your own environment setup, if you are using some other installation, then additional packages may be needed:
	* OpenIPMI-modalias.x86_64	* OpenIPMI-libs.x86_64	* libyaml.x86_64	* PyYAML.x86_64	* libesmtp.x86_64	* net-snmp-libs.x86_64	* net-snmp-agent-libs.x86_64	* openhpi-libs.x86_64	* libtool-ltdl.x86_64	* perl-TimeDate

* Extract and Install MQ 9.1.2

	The code is provided as a compressed tar file in the directory /home/student/Downloads.
	
* Install the MQ and RDQM code 

	RDQM is a single feature which now supports HA and/or DR (but not at the same time for a single queue manager). The RDQM support requires the Server and Runtime packages. 
	Run the installation script.		

* Configure the RedHat firewall 

	If there is a firewall between the nodes in the HA group, then the firewall must allow traffic between the nodes on a range of ports. 
	
	Firewall (firewalld) enabled, and ports 1500 & 1501 will be defined during the lab.
	
* Configure the OS storage settings 

	If the system uses SELinux in a mode other than permissive, you must run the following command:

	   ```
	    semanage permissive -a drbd_t
	   ``` 
	
* Configure groups 

	To create, delete, or configure replicated data queue managers (RDQMs) you must use a user ID that belongs to both the mqm and haclient groups. 
	
	If want to allow a normal user in the mqm group to create RDQM instances etc., you need to grant the user access to the certain commands via sudo. The user will also need to be part of the mqm group. 
	
	You will add the mqm user to the root and haclient group. Then add root, student, and ibmdemo to the mqm and haclient groups. 
	
	 The following groups set up: 

	* **mqm** to allow user to run specific MQ commands, 
	* A normal user "ibmdemo" has been defined for running applications and MQ commands.

	|Name   | Password |  Purpose | Group |
|:-----:|:--------:|:--------:|:-----:|
|root | passw0rd | superuser |  |
|ibmdemo | passw0rd | host vm user |mqm     |
|ibmdemo | passw0rd | MQ user |mqm     |
	
  	
* Create the Logical Group for the QM data 

	Each node requires a volume group named drbdpool. The storage for each replicated data queue manager is allocated as a separate logical volume per queue manager from this volume group. For the best performance, this volume group should be made up of one or more physical volumes that correspond to internal disk drives (preferably SSDs). 

The above steps must be completed on each node before RDQM can be configured. At this point you are ready to begin RDQM configuration. 


### Setup the RHEL image (pre-configured on SkyTap):

In the Skytap environment, there are 3 virtual machines miqmp, miqms, mqnfs4 which currently should be in a powered off or paused state.

![](./images/pots/mq-ha/lab2/image1.png)
1. Click the **run** button to resume the VMs. 

2. Click the monitor icon for *miqmp* which will launch the desktop in another browser tab.

	![](./images/pots/mq-ha/lab2/image2.png)

1. Log on to VM **miqmp** as user ibmdemo, using password passw0rd.

	![](./images/pots/mq-ha/lab2/image3.png)

## Configure RDQM-DRA primary instance of a disaster recovery queue manager is created on one server. A secondary instance of the same queue manager must be created on another server, which acts as the recovery node. Data is replicated between the queue manager instances. The replication of the data between the two nodes is handled by DRBD.
Unlike the High Availability solution, there is no heartbeat detection between the two nodes. If the primary queue manager node is lost, the secondary instance can be manually made into the primary instance, the queue manager started, and work resumed.
Data replication between primary and secondary queue managers can be done synchronously or asynchronously. If the asynchronous option is selected, operations such as PUT or GET complete and return to the application before the data is replicated to the secondary queue manager. Asynchronous replication means that, following a recovery situation, some messaging data might be lost. But the secondary queue manager will be in a consistent state, and able to start running immediately, even if it is started at a slightly earlier part of the message stream.
You will configure a DR RDQM that uses asynchronous replication.

### Create the DR RDQMYou will create a DR RDQM with asynchronous replication. You must first create a primary RDQM DR queue manager. Then you will create a secondary instance of the same queue manager on another node. The primary and secondary instances must have the same name and be allocated the same amount of storage.

1. You should be logged on as *ibmdemo* on **miqmp**.

1. Right-click on the desktop and select *Open Terminal* 

	![](./images/pots/mq-ha/lab3/image1.png)

1. Switch to root user with the command:

	```
	su -
	```
	Enter the *passw0rd* for root's password.
	
1. Enter the command to set the MQ environment.

	```
	. /opt/mqm/bin/setmqenv -s
	```
	
1.	Create a primary queue manager on node **miqmp**. It will use asynchronous replication. The local IP for DR replication is 10.0.4.1. The recovery IP used for replication on the secondary instance is 10.0.4.2. Replication will take place using port 7001. The queue manager will be DRQM1. 

	In the terminal window, create the primary node: 

	```
	crtmqm -rr p -rt a -rl 10.0.4.1 -ri 10.0.4.2 -rn miqms -rp 7001 -p 1502 DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image2.png)
	
	Notice at the end, the command needed to create the secondary instance is provided for you.
	
1. Switch to the **miqms** VM. Login as *ibmdemo / passw0rd* 

1. As you did on miqmp, open a terminal window, switch user to **root**, and issue the command to set up the MQ environment.

2. Create a secondary instance of the queue manager on node **miqms**. In the terminal window, enter the command which was **provided** for you when you ran the crtmqm command on miqmp. For example:
	```
	crtmqm -rr s -rt a -rl 10.0.4.2 -ri 10.0.4.1 -rn miqmp -rp 7001 DRQM1
	```

	![](./images/pots/mq-ha/lab3/image3.png)
	
1. Check the status on both nodes with the command to ensure they are correct. On node **miqmp** use the command: 
	
	```
	rdqmstatus -m DRQM1
	```	On **miqms**: 
	
	![](./images/pots/mq-ha/lab3/image4.png) 
	
	On **miqmp**:
	
	![](./images/pots/mq-ha/lab3/image5.png)

1. Issue the command again until it shows synchronization is complete. When initial synchronization has completed, it should look similar to the following:

	![](./images/pots/mq-ha/lab3/image6.png)

1. On the node with the secondary instance, the output should initially look similar to the following:

	![](./images/pots/mq-ha/lab3/image7.png)
	
1. On node **miqms**, open a new terminal window (as the ibmdemo user), and enter the command to set the MQ environment.	

1. On node **miqmp**, open a new terminal window (as the ibmdemo user), enter the command to set the MQ environment, and then start the queue manager with the following command:

	```
	. /opt/mqm/bin/setmqenv -s
	```
	```
	strmqm DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image7b.png)


1. Now check the status on both nodes to ensure they are correct, using the command:	
	```
	rdqmstatus -m DRQM1
	```	On **miqmp**, the output will initially look similar to the following:
	
	![](./images/pots/mq-ha/lab3/image8.png)
	
	As the node with the secondary instance only runs the queue manager when DR is needed, the output will be unchanged and look as it did previously.
	
	![](./images/pots/mq-ha/lab3/image9.png)
	
## Test the DR SecondaryNow that the DR nodes have been set up, you will test the secondary DR queue manager.### Make the Primary instance the Secondary nodeOnly one node can be the Primary. Therefore, before another node can be designated the Primary, the original Primary needs to be designated the Secondary.

1. On node **miqmp**, in ibmdemo's terminal, stop the queue manager:

	```	endmqm DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image10.png)
	
1. On node **miqmp**, in the **root** terminal, designate node miqmp, as the secondary using the rdqmdr command:	
	```	rdqmdr -m DRQM1 -s
	```
	
	![](./images/pots/mq-ha/lab3/image12.png)
		
### Make the Secondary node the Primary instanceDesignate the Secondary node as the Primary instance. 

1. On the recovery node **miqms** in the **root** terminal, designate it as the primary instance using the rdqmdr command:	
	```
	rdqmdr -m DRQM1 -p
	```
	
	![](./images/pots/mq-ha/lab3/image13.png)

1. Still on **miqms** in the root terminal or ibmdemo's terminal, start the queue manager:	```	strmqm DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image14.png)	
1. Confirm the status of both nodes:	
	```
	rdqmstatus -m DRQM1
	``` 
	
	On node **miqms**: 
	
	![](./images/pots/mq-ha/lab3/image15.png)
	
	On node **miqmp**:
	
	![](./images/pots/mq-ha/lab3/image16.png)
	
	Provided that channels were defined with a list of alternative connection names specifying the primary and secondary queue managers, then applications will automatically connect to the new primary queue manager.
	
### Make the Primary instance the Primary againIf the loss of the Primary was only temporary, you would want to designate it as the Primary again. This would be achieved as described below.

1. On node **miqms**, stop the queue manager:	
	```
	endmqm DRQM1
	```
	
1. In the **root** terminal, designate node miqms as the secondary:	
	```
	rdqmdr -m DRQM1 -s
	```
	
	![](./images/pots/mq-ha/lab3/image17.png)
		

1. On the primary node **miqmp**, in the **root** terminal, designate it as the primary instance again:	
	```
	rdqmdr -m DRQM1 -p
	```
	
1. On the primary node **miqmp** (in either root terminal or ibmdemo terminal), restart the queue manager:	
	```
	strmqm DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image18.png)
	
1. Confirm the status of both nodes:	
	```
	rdqmstatus -m DRQM1
	```
	
	On **miqmp**:
	
	![](./images/pots/mq-ha/lab3/image19.png)
	
	On **miqms**:
	
	![](./images/pots/mq-ha/lab3/image20.png)
	
## Replace node that was running a DR PrimarySuppose the loss of the primary node was due to a failure, which resulted in the node having to be replaced. You would want to replace the primary node while the queue manager runs on the secondary node. Then restore the original disaster recovery configuration.### Simulate the loss of the Primary nodeAlthough the node has not been lost, you will simulate it by disabling the DR Replication Network adapter and deleting the queue manager. 

1. On node **miqmp**, go to **Applications -> System Tools -> Settings**:

	![](./images/pots/mq-ha/lab3/image21.png)

1. On the left list of entries, scroll down and select **Network**:

	![](./images/pots/mq-ha/lab3/image22.png)

1. Click the **Settings** gear symbol on the **ens38** network adapter, to verify that it is the DR Replication adapter (IP address 10.0.4.1):

	![](./images/pots/mq-ha/lab3/image22a.png)
	
1. In the window that opens, validate the IP address of **10.0.4.1**, then click **Cancel**:

	![](./images/pots/mq-ha/lab3/image22b.png)
	
1. For the **ens38** adapter, click the button to switch it off.

	![](./images/pots/mq-ha/lab3/image23.png)

1. On node **miqmp**, stop the queue manager:	
	```
	endmqm DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image24.png)
	
1. In the **root** terminal, remove the queue manager:	
	```
	dltmqm DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image25.png)
	
### Make the Secondary instance the PrimaryDesignate the Secondary node as the Primary instance. 

1. On the recovery node **miqms**, in the **root** terminal, designate it as the primary instance:	
	```
	rdqmdr -m DRQM1 -p
	``` 
	
1. Start the queue manager:	
	```
	strmqm DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image26.png)
	
1. Confirm the status of both nodes: 

	```
	rdqmstatus -m DRQM1
	```
	
	On **miqms**:
	
	![](./images/pots/mq-ha/lab3/image27.png)
	
	As there is no longer a queue manager defined on node **miqmp**, the output should look similar to the following:
	
	![](./images/pots/mq-ha/lab3/image28.png)
	
### Add the new Primary node into the DR configurationFor the replacement node to be brought back into the DR configuration, it must assume the identity of the failed node -- the name and IP address must therefore be the same. 

1. Remember, *rdqmdr* commands require root authority. You will determine the command that needs to be run on the new Primary node. On node **miqms**, run the command :	
	```
	rdqmdr -m DRQM1 -d
	```
	
	The output should look similar to the following: 
	
	![](./images/pots/mq-ha/lab3/image29.png)
	
1. On node **miqmp**, restart the DR Replication network interface. Go to **Applications -> System Tools -> Settings -> Network**, and click the button to turn on **ens38**. 

	![](./images/pots/mq-ha/lab3/image30.png)

1. Copy the command (as highlighted above) into the command line of the new Primary node, **miqmp**, to run it:	
	```
	crtmqm -rr s -rl 10.0.4.1 -ri 10.0.4.2 -rn miqms -rp 7001 DRQM1
	```
		The output should look similar to the following:
	
	![](./images/pots/mq-ha/lab3/image31.png)
	
1. Check the status of the synchronization on both nodes:	
	```
	rdqmstatus -m DRQM1
	```
	
	On **miqmp**:
	
	![](./images/pots/mq-ha/lab3/image32.png)
	
	On **miqms**:
	
	![](./images/pots/mq-ha/lab3/image33.png)

### Restore the original DR configurationTo restore the original DR configuration, you would want to designate the Primary node as the Primary instance again.

1. When the initial synchronization is complete on the primary node miqmp, you can designate the secondary node as the secondary instance again. End the queue manager on node **miqms**:	
	```
	endmqm DRQM1
	```
	
1. Designate node miqms as the secondary instance again:	
	```
	rdqmdr -m DRQM1 -s
	```
	
	![](./images/pots/mq-ha/lab3/image34.png)
	
1. On node miqmp, designate it as the primary instance again:	
	```
	rdqmdr -m DRQM1 -p
	```
	
1. Start the queue manager on the primary node:	
	```
	strmqm DRQM1
	```	
	
	![](./images/pots/mq-ha/lab3/image35.png)
		
1. Confirm the status of both nodes:	
	```
	rdqmstatus -m DRQM1
	```
	
	On **miqmp**:
	
	![](./images/pots/mq-ha/lab3/image36.png)
	
	On **miqms**:
	
	![](./images/pots/mq-ha/lab3/image37.png)

	
## Replace node that was running a DR SecondaryIf it is a secondary node that needs to be replaced, you would just replace it and restore it to the original disaster recovery configuration.### Add the new Secondary node into the DR configurationTo simulate this, there is no need to change the DR designations, prior to the Secondary node being replaced. You will simply disable the DR Replication Network adapter.
For the replacement node to be brought back into the DR configuration, again it must assume the identity of the failed node -- the name and IP address must be the same.

1. On node **miqms**, go to **Applications -> System Tools -> Settings**.

	![](./images/pots/mq-ha/lab3/image38.png)

	Select **Network**. 
	
	![](./images/pots/mq-ha/lab3/image39.png)
	
1. The DR Replication adapter (IP address 10.0.4.2) is the **ens38** adapter. Click the button to switch it off. 

	![](./images/pots/mq-ha/lab3/image40.png)

1. In **root's** command window, delete the queue manager.

	```
	dltmqm DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image40a.png)

1. You will assume the secondary node has been replaced. Determine the command that needs to be run on the new Secondary node. On node **miqmp**, run the command :	
	```
	rdqmdr -m DRQM1 -d
	```
	
	![](./images/pots/mq-ha/lab3/image41.png)
	
1. On node **miqms**, go to **Applications -> System Tools -> Settings -> Network**. Click the button for the DR Replication adapter **ens38** (IP address 10.0.4.2), to switch it back on.

	![](./images/pots/mq-ha/lab3/image42.png)

1. Copy this command, that was displayed, into the command line of the new Secondary node, **miqms**, then run it:	
	```
	crtmqm -rr s -rl 10.0.4.2 -ri 10.0.4.1 -rn miqmp -rp 7001 DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image43.png)
			
1. Confirm the status of the DR configuration on both nodes:	
	```
	rdqmstatus -m DRQM1
	```
	
	On **miqms**:
	
	![](./images/pots/mq-ha/lab3/image45.png)
	
	On **miqmp**:
	
	![](./images/pots/mq-ha/lab3/image46.png)
	
## Reverting to a snapshotSuppose a network connection between the nodes is lost; the changes to the persistent data for the primary instance of a queue manager are tracked. When the network connection is restored, a synchronization process is used to get the secondary instance up to speed as quickly as possible.
While synchronization is in progress, the data on the secondary instance is in an inconsistent state. A snapshot of the state of the secondary queue manager data is taken.If a failure of the main node or the network connection occurs during synchronization, it would be necessary to revert the secondary instance back to this snapshot.

### Create a snapshotYou will first create an inconsistent state on your DR nodes.

1. On **miqmp**, open a new terminal (as ibmdemo), and run the command to set the MQ environment.

	```
	. /opt/mqm/bin/setmqenv -s
	```
	
1. Create a local queue for placing messages to provide some data for later synchronization. Use the runmqsc command to create a local, persistent queue, called Q1DR.

	```
	runmqsc DRQM1
	```
	
	```
	DEFINE QLOCAL(Q1DR) DEFPSIST(YES)
	```
	
	```
	end
	```
	
	![](./images/pots/mq-ha/lab3/image47.png)

1. Check the status on both nodes is normal.	
	```
	rdqmstatus -m DRQM1
	```
	
1. Get ready to simulate a failure of the DR replication network adapter.On node **miqmp**, open a new window, as user *root / passw0rd*. Use the iptables rules to remove all output packets from the network adapter interface.
	
	Type the following command or copy / paste, but **DO NOT PRESS ENTER YET** - the rule will be applied BEFORE all the messages have been put on the queue. You will be instructed when to press enter in root's terminal window.
	
	
	```
	iptables -I OUTPUT -o ens38 -j DROP
	```
	
1. In the user ibmdemo window, start putting some messages onto the queue. Use the amqsblst sample to do this:	
	```
	cd /opt/mqm/samp/bin
	```
		```
	./amqsblst DRQM1 Q1DR -W -s 1000 -c 5000
	```
		The output should look similar to the following:
	
	
	![](./images/pots/mq-ha/lab3/image48.png)

	
1. Before all the messages have been put onto the queue, in the user **root** window,press **<enter>** to add the iptables rule. This will simulate a network outage on node miqmp.
	There will be a short pause, then the placing of messages will resume.
	
	![](./images/pots/mq-ha/lab3/image49.png)

	
1. In another window as user ibmdemo, check the status on both nodes	
	```
	rdqmstatus -m DRQM1
	```
		Notice on node **miqmp**, the DR status is showing as ‘Remote unavailable’.
	
	![](./images/pots/mq-ha/lab3/image51a.png)
	
	Similarly, on node **miqms**:
	
	![](./images/pots/mq-ha/lab3/image51.png)
	
1. Issuing the command again on node **miqmp**, when all the messages have been placed on the queue, you will notice the ‘DR out of sync data’ has changed. Your number will be different than the screenshot.
	
	![](./images/pots/mq-ha/lab3/image50.png)

1. Simulate the restoration of the network outage on node **miqmp** by issuing an iptables remove previous rule command. In the user **root** window:	
	```
	iptables -D OUTPUT -o ens38 -j DROP
	```
	
1. The nodes will start synchronizing as soon as this happens. Check the status on both nodes immediately, before switching off the network. They will look similar to the following:	
	```
	rdqmstatus -m DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image52.png)

	
	![](./images/pots/mq-ha/lab3/image53.png)	
1. Immediately (before synchronization is *complete*), simulate a network outage on node **miqmp** again.	
	```
	iptables -I OUTPUT -o ens38 -j DROP
	```
	
1. When the network is detected to have failed again, the status on the primary node, **miqmp** goes back to ‘Remote unavailable’.
	
	![](./images/pots/mq-ha/lab3/image81.png)

	The status on the secondary node, **miqms** is ‘Inconsistent’.
	
	
	![](./images/pots/mq-ha/lab3/image54.png)
	
	There is also an indication of the amount of data that is out of synchronization on both nodes.
	
### Revert to a snapshotYou will now see how the secondary instance reverts to its snapshot and the queue manager data. Note any updates that have happened since the original network failure, however, will be lost.

1. The assumption now is that the Primary node is no longer usable, so the replication node must be made the new Primary instance. On the secondary node **miqms**, in the **root** user terminal, designate **miqms** as the primary instance:	
	```
	rdqmdr -m DRQM1 -p
	```
	
1. Due to its former ‘Inconsistent’ state, miqms will revert to a snapshot. Check the status to confirm this.	
	```
	rdqmstatus -m DRQM1
	```
	
	The output should look like the following:
	
	![](./images/pots/mq-ha/lab3/image55.png)

1. When node miqms has completed reverting to the snapshot. Checking the status again.

	```
	rdqmstatus -m DRQM1
	```
	
	should look similar to the following:
	
	
	![](./images/pots/mq-ha/lab3/image56.png)
	
	Notice the status indicates the queue manager ‘*Ended unexpectedly*', and there is data that is out of synchronization.
	
1. As this situation would have occurred as a result of a possible failure of the Primary node, in reality you would go through the process described earlier to *Add the new Primary node into the DR configuration*.

	Once the new Primary node was part of the DR configuration again, you would follow the steps to ‘Restore the original DR configuration’ of the Primary node, being the primary instance of the queue manager.
	
	Here you will simulate something similar. You will start the queue manager on node miqms. You will delete the queue manager on node miqmp, and go through the latter of the steps mentioned above again, to show some additional screens not seen previously.1. Start the queue manager on node **miqms**, which is now the primary instance.	
	```
	strmqm DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image57.png)
	
1. Confirm the queue manager on node **miqms** is running as the primary instance by checking the status.	
	```
	rdqmstatus -m DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image58.png)

	Notice that as a result of reverting to a snapshot there is data out of synchronization.

1. ‘Simulate’ the replacement of node miqmp. On node **miqmp**, in the user **ibmdemo** window, stop the queue manager:	
	```
	endmqm DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image59.png)
	
1. Confirm the queue manager on node **miqmp** ended normally by checking the status	
	```
	rdqmstatus -m DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image60.png)
	
1. In the **root** terminal, delete the queue manager on node **miqmp**.	
	```
	dltmqm DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image61.png)
	
1. On node **miqmp**, as user **root**, restart the network interface.	
	```
	iptables -D OUTPUT -o ens38 -j DROP
	```
	
1. On node **miqms**, enter the command to determine the command needed to recreate the queue manager on node miqmp.	
	```
	rdqmdr -m DRQM1 -d
	```
	
	![](./images/pots/mq-ha/lab3/image62.png)

1. On node **miqmp**, as user **root**, issue the command to recreate the queue manager:	
	```
	crtmqm -rr s -rl 10.0.4.1 -ri 10.0.4.2 -rn miqms -rp 7001 DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image63.png)
	
1. Check the status on both nodes:	
	```
	rdqmstatus -m DRQM1
	```
	
	On **miqmp**:
	
	![](./images/pots/mq-ha/lab3/image64.png)
	
	On **miqms**:
	
	![](./images/pots/mq-ha/lab3/image65.png)
	
	Notice that synchronization of data is taking place between the nodes. Indications are given on its progress and estimated completion time.
	
1. When data synchronization has completed the status of the nodes will look similar to the following:

	On **miqmp**:
	
	![](./images/pots/mq-ha/lab3/image66.png)
	
	On **miqms**:
	
	![](./images/pots/mq-ha/lab3/image67.png)
	
1. Restore the DR configuration. On node **miqms**, as user **root**, stop the queue manager:	
	```
	endmqm DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image68.png)
	
1. Make node **miqms** the secondary instance:	
	```
	rdqmdr -m DRQM1 -s
	```
	
	![](./images/pots/mq-ha/lab3/image69.png)
	
1. On node **miqmp**, as user **root**, make it the primary instance of the queue manager:	
	```
	rdqmdr -m DRQM1 -p
	```
	
	![](./images/pots/mq-ha/lab3/image70.png)

1. Start the queue manager on node **miqmp**:	
	```
	strmqm DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image71.png)

1. Confirm the DR configuration by checking the status on both nodes:	
	```
	rdqmstatus -m DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image72.png)
	
	![](./images/pots/mq-ha/lab3/image73.png)

## Delete a DR RDQMIf Disaster Recovery is no longer required for a queue manager, the queue manager needs to be deleted to be removed from the DR configuration. This is achieved as follows:

1. On **miqmp** stop the queue manager:	
	```
	endmqm DRQM1
	```
	
1. View the status to confirm that has happened :	
	```
	rdqmstatus -m DRQM1
	```
	
	![](./images/pots/mq-ha/lab3/image75.png)

1. On **miqmp** remove the queue manager:	
	```
	dltmqm DRQM1
	```
	
1. Viewing the status:	
	```
	rdqmstatus -m DRQM1
	```
	
	will confirm the queue manager no longer exists on the primary node.
	
	![](./images/pots/mq-ha/lab3/image76.png)
	
1. Also remove the queue manager on **miqms**:	
	```
	dltmqm DRQM1
	```

1. Again, viewing the status:	
	```
	rdqmstatus -m DRQM1
	```
		will confirm the queue manager no longer exists on the secondary node either.
	
	![](./images/pots/mq-ha/lab3/image77.png)
	
	
## CONGRATULATIONS! ### You have completed this hands-on lab.You have created replicated data queue managers to provide disaster recovery for IBM MQ. 
