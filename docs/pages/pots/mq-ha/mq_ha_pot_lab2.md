---
title: Replicated Data Queue Managers (RDQM) for High Availability 
toc: false
sidebar: labs_sidebar
folder: pots/mq-ha
permalink: /mq_ha_pot_lab2.html
summary: Replicated Data Queue Managers (RDQM) for High Availability
applies_to: administrator
---

# Introducing IBM MQ Replicated Data Queue Manager

## IBM MQ High Availability Options

For many years there has been two well established options for IBM MQ single site high availability: 

* **Multi-Instance Queue Managers**: 
An out of the box technology that requires a shared disk (such as NFS) for storing the Queue Manager data, and two severs in an active / standby configuration. If the active instance was to fail, the lock on the shared disk would be released, and the passive instance promoted to the active instance. 

* **Operating System High Availability technologies**: 
IBM MQ explicitly supports a number of well-established operating system high availability technologies, such as Microsoft Cluster Service, PowerHA for AIX, Veritas Cluster Server, HP Serviceguard, or a Red Hat Enterprise Linux cluster with Red Hat Cluster Suite.

Each of the above have technical considerations and dependencies; for instance, the multi-instance queue manager is dependent on a shared disk, which must be redundant to provide effective high availability; while the operating system approach requires deep knowledge of the operating system, and in certain cases additional license entitlement for the operating system.

IBM MQ V9.0.4 Continuous Delivery Release provided a new option to complement the existing two, called MQ Replicated Data Queue Manager (RDQM). This technology is based on the MQ Appliance high availability capabilities, and is available on the Linux operating system. In the simplest of situations, it will include three standard RedHat Linux servers, one being active handling requests, with the other two replicating the data and waiting to become active in a similar logical manner to the standby instance within a Multi-Instance Queue Manager. When comparing to multi-instance, there are three key differences:

* **No shared disk**: 
 Each individual server includes a complete replica of the Queue Manager data, removing the need for a shared disk, which simplifies the configuration, and potentially improves the performance of the overall solution.
    
* **Quorum based promotion**: 
 Due to each server including a complete replica, in the case of a network failure between the servers, all could be promoted to be active, and a condition called “split brain” could occur. To avoid this from happening, three servers are part of the RDQM HA group, and a server will only continue to be active or promoted to the active instance if a quorum of the servers within the RDQM HA group are contactable.
 
* **Floating IP**: 
 With Multi-Instance Queue Managers, two servers, with separate IP addresses could be hosting and running the active instance of the Queue Manager. This means that the client needs to be explicitly aware of the two IP addresses. With the new RDQM, a floating IP address is used, and associated with the server that is running the active queue manager. This means that clients only need to be aware of the floating IP address, simplifying the logic on the client.

Although the above are all drivers of why to consider the new capability, there are some points to consider when evaluating if this new HA model is suitable for a client:

* **Restricted to a single Data Center**: 
 The replication between instances within a RDQM HA group is synchronous, and therefore if the servers are not co-located within the same data center, this can affect the performance of the solution. Also, since a floating IP address is used, this normally is limited to a single data centre.
 
* **Distributed Writes**: 
 As mentioned above, the writing is completed using a synchronous write, and therefore involves writing to multiple disks across multiple servers. The performance characteristics of this approach, compared to the other options, need to be evaluated.
* **Dedicated Logical Volume Group**: 
 Each RedHat Linux server requires a dedicated logical volume for the RDQM data. This normally means a separate disk associated with the VM or bare metal machine dedicated to the RDQM data. If the server is being specially provisioned for IBM MQ, this is unlikely to be a major issue.
* **Split Brain**: 
 The quorum based approach to electing primaries greatly reduces the chances of a split brain situation occurring, but does not completely remove all edge cases.
* **RedHat Enterprise Linux 7.3, 7.4, 7.5, or 7.6**: 
 The RDQM capability is currently supported on RedHat Enterprise Linux 7.3, 7.4, 7.5 and 7.6. Depending on the environment, this may or may not be a significant restriction.

 
To assist with the setup of RDQM, we’ve created a number of scripts and helper files. The following is based on the process required for a base RedHat Enterprise Linux 7.6 install.

## Lab Introduction

This lab provides a demonstration of a new approach to High Availability in MQ on Linux, with the following key features:

* Use of Distributed Replicated Block Device (DRBD) storage rather than network shared storage* Use of a cluster resource manager (Pacemaker) to manage a cluster of three nodes* A new kind of queue manager called a Replicated Data Queue Manager (RDQM):	* a RDQM is active on only one node at any one time	* each node can run different active RDQMs	* each RDQM has a preferred location (node) in normal operation	* a quorum prevents an RDQM from running on more than one node at the same time	* a RDQM can have a floating IP address associated with it to simplify configuration of clients and other queue managers that need to communicate with the RDQM

### Summary of lab environment

* Three RHEL 7.6 x86_64 systems running in Skytap:
	* miqmp	-	This will be our primary node.	* miqms	-	This will be a secondary node,	* mqnfs4	-	This will be another secondary node.

* VMWare Workstation virtual networks: 

	|Name   | Type | SkyTap Network |  Subnet  | DHCP |
	|:-----:|:--------:|:--------:|:-----:|:-----:|
	|VMnet1 | host-only | HA1 | 10.0.1.0 |no     |
	|VMnet2 | host-only | HA2 | 10.0.2.0 |no     |
	|VMnet3 | host-only | HArep | 10.0.3.0 |no     |
	|VMnet4 | host-only | DRrep | 10.0.4.0 |no     |
	|VMnet8 | NAT | Network 1 | 10.0.0.0 |no     |

* Network interfaces:

	|Interface Purpose | Interface Name |  miqmp (Primary node)  | miqms (Secondary node) | mqnfs4 (Secondary node) |
	|:------:|:--------:|:--------:|:-----:|:--------|
	|MQ Fixed IP | ens34   | 10.0.0.1 |10.0.0.2 |10.0.0.3 |
	|MQ Floating IP | |10.0.0.10 | 10.0.0.10 |10.0.0.10|

	HA interfaces are used as follows: 
	
	* HA Primary - to monitor the nodes in the cluster	* HA Alternate - backup for monitoring the cluster if the HA Primary network fails	* HA Replication - for synchronous data replication (the higher the bandwidth the better and the lower the latency the better)	**Note**:  Hosts miqmp, miqms, mqnfs4 are tied to the Administration IP addresses above.
* Dedicated volume group "drbdpool" containing a single physical volume on each node for RDQM, but please note, you will not see any further reference to this in this document.
* The following groups configured: 

	* **mqm** to allow user to run specific MQ commands 
	* **haclient** to allow user to run HA-specific commands
* A normal user "ibmdemo" has been defined for running applications and MQ commands.

	|Name   | Password |  Purpose | Group |
|:-----:|:--------:|:--------:|:-----:|
|root | passw0rd | superuser |  |
|ibmdemo | passw0rd | host vm user |mqm     |
|ibmdemo | passw0rd | MQ user |mqm     |
* Firewall (firewalld) enabled, and ports 1500 & 1501 will be defined during the lab.
* The following Pacemaker dependencies have already been installed. This list should be sufficient for a standard installation of RHEL 7.6 Server or Workstation. For your own environment setup, or if you are using some other installation, additional packages may be needed:
	* OpenIPMI-modalias.x86_64	* OpenIPMI-libs.x86_64	* libyaml.x86_64	* PyYAML.x86_64	* libesmtp.x86_64	* net-snmp-libs.x86_64	* net-snmp-agent-libs.x86_64	* openhpi-libs.x86_64	* libtool-ltdl.x86_64	* perl-TimeDate

Depending on your security configuration, there are three different ways to configure the RDQM feature:
1. The simplest way is if the mqm user can ssh between the three nodes of the cluster without a password and can sudo to run the necessary commands.
2. The intermediate option is if the mqm user can sudo but not ssh. It is preferable if the actual users are also in the haclient group. 
3. The default is that the mqm user cannot ssh or sudo. In this lab, instructions are provided to setup and test using the intermediate method. 

## Setup the RHEL image (pre-configured on SkyTap):

In the Skytap environment, there are 3 virtual machines: miqmp, miqms, and mqnfs4, which currently should be in a powered off or paused state.

![](./images/pots/mq-ha/lab2/image1.png)
1. Click the **run** button to start or resume the VMs. 

1. Click the monitor icon for *miqmp* which will launch the desktop in another browser tab.

	![](./images/pots/mq-ha/lab2/image2.png)

1. Log on to VM **miqmp** as user ibmdemo, using password passw0rd. 

	![](./images/pots/mq-ha/lab2/image3.png)### Pre-configuration steps 
The following steps are necessary for configuring RDQM, and are shown for your reference. They have **already been completed** on the VMs. 

* Extract and Install MQ 9.1.2 

	The code is provided as a compressed tar file in the directory /home/ibmdemo/.
	
* Install the MQ and RDQM code 

	RDQM is a single feature which now supports HA and/or DR (but not at the same time for a single queue manager). The RDQM support requires the Server and Runtime packages. 
	Run the installation script.		

* Configure the RedHat firewall 

	If there is a firewall between the nodes in the HA group, then the firewall must allow traffic between the nodes on a range of ports. Open another terminal, switch to user root, and run the sample file.

* Configure the OS storage settings 

	If the system uses SELinux in a mode other than permissive, you must run the following command:

	   ```
	    semanage permissive -a drbd_t
	   ``` 
	
* Configure groups 

	To create, delete, or configure replicated data queue managers (RDQMs), you must use a user ID that belongs to both the mqm and haclient groups. 
	
	If you want to allow a normal user in the mqm group to create RDQM instances etc., you need to grant the user access to certain commands via sudo. 	
	You will add the mqm user to the root and haclient group. Then add root and ibmdemo to the mqm and haclient groups. 
  	
* Create the Logical Group for the QM data 

	Each node requires a volume group named drbdpool. The storage for each replicated data queue manager is allocated as a separate logical volume per queue manager from this volume group. For the best performance, this volume group should be made up of one or more physical volumes that correspond to internal disk drives (preferably SSDs). 
	
The above steps must be completed on each node before RDQM can be configured. At this point, you are ready to begin RDQM configuration. 

## Configure RDQM### Configure the firewallNormally, the firewall would have been configured as a pre-req. However during preparation of this environment, the default RHEL firewall was not configured. You need to do that now for the RDQM cluster. 

1. On **miqmp** open a new terminal window and switch user to root using the following command:

	```
	su -
	```
	
	When prompted, enter root's password *passw0rd*. 
	
1. Start the firewall with following command:

	```
	systemctl start firewalld
	```

1. Run the following command to allow MQ, DRDB, and Pacemaker ports opened in the firewall:

	```
	/opt/mqm/samp/rdqm/firewalld/configure.sh
	```
	
	![](./images/pots/mq-ha/lab2/image000.png)
	
1. Exit from the root user.	

	```
	exit
	```

**Repeat the above commands** on **miqms** and **mqnfs4** to configure the firewall on each of those VMs.

**Hint:**  You can click the monitor icon for miqms and mqnfs, which will launch the desktop for each in a new browser tab.


**Be sure to exit out of root before continuing with the next section.**
### Configure the clusterThe cluster must first be created, and then an RDQM instance defined containing one or more queue managers. The RDQM code expects the rdqm.ini file to be in the /var/mqm directory.The cluster is defined using the rdqm.ini file. The /home/ibmdemo/mq912 directory contains the rdqm.ini file we will use.     

![](./images/pots/mq-ha/lab2/image4.png)

	
1. On node **miqmp**, in a terminal window (as **ibmdemo**), navigate to **/var/mqm**. 

1. Copy the provided **rdqm.ini** file to the **/var/mqm** directory with the following command: 

    ```
    cd /var/mqm
    sudo cp /home/ibmdemo/mq912/mqScripts/rdqm.ini . 
    ```
    
    ![](./images/pots/mq-ha/lab2/image5.png)
    
1. **IMPORTANT:** Repeat these commands on **miqms** and **mqnfs4** before continuing.

1. Return to **miqmp** and enter the command to configure RDQM on the primary machine:

	```
    sudo /opt/mqm/bin/rdqmadm -c 
   ```
    Enter ibmdemo's password when prompted, passw0rd.
    
    ![](./images/pots/mq-ha/lab2/image6.png)
    
1. You have received the messages that the replicated data system has been completed on this node. You are instructed to run the "rdqmadm -c" on the other two nodes.

1. Enter the command on **miqms**.

	  ![](./images/pots/mq-ha/lab2/image7.png)
	  
1. Enter the command on **mqnfs4**. Notice the message that reports configuration is complete.

	![](./images/pots/mq-ha/lab2/image8.png) 
	
### Configure the HA RDQM

The high availability replicated data queue manager (RDQM) now needs to be created. Secondary RDQMs need to be created on two of the nodes, and then the primary RDQM needs to be created on the node that will run the RDQM by default (its preferred location).

You will start by creating the secondary RDQM on node 2 (miqms).
1. Switch to node **miqms**. 
	In a terminal window, switch to the root user and enter the password passw0rd when prompted. 
	
1. Set the MQ environment with the following command:

	```	. /opt/mqm/bin/setmqenv -s
	```

1. Create a secondary queue manager called **myRDQM** with the following command:	```	crtmqm -sxs myRDQM
	```
	
	![](./images/pots/mq-ha/lab2/image9.png)	The secondary queue manager has been created.
	
1. Switch to the **mqnfs4** node and enter the same commands to create the secondary queue manager there.

1. Verify that the secondary queue manager has been created. 

	![](./images/pots/mq-ha/lab2/image10.png)
	
1. Return to node **miqmp**. In a terminal window, switch to the root user and enter the password passw0rd when prompted. 

1. Set the MQ environment with the following command:

	```	. /opt/mqm/bin/setmqenv -s
	``` 

1. You will create the primary RDQM on node 1. Create the primary RDQM, which will listen on port 1500 with the following command: 

	```
	crtmqm -p 1500 -sx myRDQM
	```
	
	![](./images/pots/mq-ha/lab2/image11.png)
	
	The primary queue manager is now created and running. 1. Check the status of the queue manager:

	```
	rdqmstatus -m myRDQM
	```
	Initially, the output should look similar to the following. Synchronization is in progress. 
	
	![](./images/pots/mq-ha/lab2/image12.png)
	
	You can run the same command on the other nodes and get similar output.
	
	![](./images/pots/mq-ha/lab2/image13.png)
	![](./images/pots/mq-ha/lab2/image14.png)
	
1. When the nodes have completed synchronising, the HA Status field on all nodes should change to ‘Normal’. This may take a few minutes. 

	Repeat the **rdqmstatus** command again until you see the normal HA status (as shown on **miqmp**).

	![](./images/pots/mq-ha/lab2/image15.png)

	
	{% include note.html content="RHEL default time before the screen locks is very short. If you need longer, you can turn off the screen lock in settings. Applications \> System Tools \> Settings \> Power \> Power Saving \> Blank screen \> Never." %}
	
	
	
## Simple testing of RDQMOnce all nodes have an HA status of Normal, you can commence testing. You will perform some tests, which will show different use cases.
### Failing over an RDQM instance to another nodeThe easiest way to force an RDQM instance to fail over to another node is to change its preferred location.

The default location for RDQM is miqmp. You will fail the RDQM instance to node 2, miqms. 

1. Switch to **miqms**. 

1. Open a new terminal window, and as the user ibmdemo, issue the command to set the MQ environment. Make this node the primary instance with the following command:	```
	rdqmadm -m myRDQM -p
	```
	
	![](./images/pots/mq-ha/lab2/image16a.png)
	
1. Confirm that **miqms** is now the primary node:

	```
	rdqmstatus -m myRDQM
	```
	
	![](./images/pots/mq-ha/lab2/image17.png)
	
1. Now you can move the queue manager back to miqmp. Return to **miqmp** and run the **rdqmadm** command again. (If you open a new terminal, don't forget to set the environment.)

	```
	rdqmadm -m myRDQM -p
	```
	
	![](./images/pots/mq-ha/lab2/image18.png)
	
	Check the status again to see it is now running on **miqmp**.
	
	```
	rdqmstatus -m myRDQM
	```
	
	![](./images/pots/mq-ha/lab2/image19.png)

### 	Move the RDQM by suspending a nodeAnother test is to move a RDQM by suspending the node on which it is running, as you may want to do when applying a Fix Pack.

1. On the node where myRDQM is running (**miqmp**), return to ibmdemo's terminal window. As the user **ibmdemo** (not as root), issue the command to suspend the queue manager:
	
	```
	rdqmadm –s
	```

	**Hint:** You may need to run the **setmqenv** command again as the ibmdemo user.

1. As shown in the display, the replicated data node is suspended and goes into standby.

	![](./images/pots/mq-ha/lab2/image20.png)
	
1. Still on **miqmp**, issue the command to resume the replicated data node in the cluster. 

	```
	rdqmadm -r
	```
	
	Quickly run the status command again. myRDQM will initially run in a secondary role on this node. If you aren't quick enough, you may not catch this transitory state.  
	
	![](./images/pots/mq-ha/lab2/image21.png)
	
1. After the node has fully resumed, myRDQM will run in a primary role on this node, as it was prior to being suspending. Issue the status command again to confirm that this has indeed happened.

	```
	rdqmstatus -m myRDQM
	```
	
	![](./images/pots/mq-ha/lab2/image22.png)

## Testing RDQM using HA sample programsSome High Availability sample programs are provided with MQ, which are a good visible demonstration for testing failovers. You will use these for testing:
* **amqsphac** - puts a sequence of messages to a queue with a two second delay between each message and displays events sent to its event handler. This will run on **mqnfs4**.* **amqsmhac** - copies messages from one queue to another with a default wait interval of 15 minutes after the last message that is received before the program finishes. This will run on **miqms**.* **amqsghac** - gets messages from a queue and displays events sent to its event handler. This will run on **miqmp**.### Create MQ resourcesTo run these samples, you will define two queues: for *SOURCE* and *TARGET*. You will also create a new channel using the ‘MQ’ IP address for each of the three nodes in our cluster (as the queue manager could run on any one of them) and the listener port for the queue manager. You will turn off CHLAUTH and CONNAUTH completely to keep things simple.

1. On **miqmp** where the myRDQM queue manager is running, in ibmdemo's terminal window, run the command:

	```
	runmqsc myRDQM
	```
	
	Run the following MQSC commands (remember that MQ objects are case sensitive):
	
	```
	ALTER QMGR CHLAUTH(DISABLED) CONNAUTH(' ')
	```
	```
	REFRESH SECURITY TYPE(CONNAUTH)
	```
	```
	DEFINE QLOCAL(SOURCE) DEFPSIST(YES)
	```
	```
	DEFINE QLOCAL(TARGET) DEFPSIST(YES)
	```
	```
	DEFINE CHANNEL(CHANNEL1) CHLTYPE(SVRCONN) TRPTYPE(TCP)
	```
	```
	DEFINE CHANNEL(CHANNEL1) CHLTYPE(CLNTCONN) TRPTYPE(TCP) CONNAME('10.0.0.1(1500),10.0.0.2(1500),10.0.0.3(1500)') QMNAME(myRDQM)
	```
	```
	END
	```
	
	![](./images/pots/mq-ha/lab2/image23.png)
	
1. On each of the nodes, open the firewall port defined. Open the firewall from the top left of the screen, under Applications -> Sundry -> Firewall.

	![](./images/pots/mq-ha/lab2/rdqm-image87.png)
	
1. Enter the password for ibmdemo, passw0rd, then click Authenticate.

	![](./images/pots/mq-ha/lab2/image24.png)
	
1. In the Ports pane, add TCP ports **1500** and **1501** (the latter will be used later).

	![](./images/pots/mq-ha/lab2/image25.png)
	
	Results should look like this:
	
	![](./images/pots/mq-ha/lab2/image26.png)
	

**Don't forget to do this on all three nodes.**
	
### Start the HA sample programsThe easiest way to configure access to the queue manager from the sample programs is to use the MQSERVER environment variable. Again, as there are 3 possible nodes where our queue manager could run, each needs to be specified, along with the listener port for the queue manager. 

1. On **miqmp**, in the user ibmdemo terminal window, enter:	
	```
	export MQSERVER='CHANNEL1/TCP/10.0.0.1(1500),10.0.0.2(1500),10.0.0.3(1500)'
	```
	
1. Change to the **/opt/mqm/samp/bin** directory, and run the command: **amqsghac TARGET myRDQM**	
	```
	cd /opt/mqm/samp/bin
	./amqsghac TARGET myRDQM
	```

	![](./images/pots/mq-ha/lab2/image27.png)
	
	Later, this will display the messages generated by amqsphac on mqnfs4.	**Leave this command to run!**
	
1. Now switch to **miqms**. In the user ibmdemo terminal window, enter:
	
	```
	export MQSERVER='CHANNEL1/TCP/10.0.0.1(1500),10.0.0.2(1500),10.0.0.3(1500)'
	```
1. Change directory to **/opt/mqm/samp/bin** and run the command: **amqsmhac -s SOURCE -t TARGET -m myRDQM**


	```
	cd /opt/mqm/samp/bin
	./amqsmhac -s SOURCE –t TARGET –m myRDQM
	```		![](./images/pots/mq-ha/lab2/image28.png)
	
	**Leave this command to run!**

1. Now switch to **mqnfs4**. As before open a new terminal window. As the user ibmdemo enter:	```
	export MQSERVER='CHANNEL1/TCP/10.0.0.1(1500),10.0.0.2(1500),10.0.0.3(1500)'
	```	
1.	Change directory to **/opt/mqm/samp/bin** and run the command: **amqsphac SOURCE myRDQM**

	```
	cd /opt/mqm/samp/bin
	./amqsphac SOURCE myRDQM
	```	
		![](./images/pots/mq-ha/lab2/image29.png)
	
	**Leave this command to run!**

1. Confirm that these messages are also being displayed on **miqmp**.

	![](./images/pots/mq-ha/lab2/image30.png)
	
**Note:** At this stage, the queue manager is running on the primary node (miqmp) and each sample program is able to communicate with it, using the first location specified in the MQSERVER environment variable:

CHANNEL1/TCP/**10.0.0.1(1500)**,10.0.0.2(1500),10.0.0.3(1500)

### Move the RDQM

You will now use the approach of controlling where the RDQM runs by changing its preferred location, in this case to miqms.1. Switch to **miqms**. In a new terminal window, run the following command as ibmdemo:
	```
	. /opt/mqm/bin/setmqenv -s
	rdqmadm –m myRDQM –p
	```
	1. Check that the queue manager is indeed running on **miqms**, by running:

	```
	rdqmstatus –m myRDQM
	```
	![](./images/pots/mq-ha/lab2/image31.png)
		The output from the amqsmhac command, running in another window, should now be like this: 
	
	![](./images/pots/mq-ha/lab2/image32.png)

1. Now switch to **miqmp**.
	Messages should continue to be received without loss, by amqsghac, after it connects with the queue manager at the new location:

	![](./images/pots/mq-ha/lab2/image33.png)
	
1. Now switch to **mqnfs4**.
	The output from the amqsphac command, running on mqnfs4, should similarly show messages continuing to be sent without loss:
	
	![](./images/pots/mq-ha/lab2/image34.png)
	
	**Note:** Now the queue manager is running on miqms, but each sample program is still able to communicate with it, this time using the second location specified in the MQSERVER environment variable:
	
	CHANNEL1/TCP/10.0.0.1(1500),**10.0.0.2(1500)**,10.0.0.3(1500) 
	
## HA Sample programs with RDQM & Floating IP addressIt is possible to associate a floating IP address with an RDQM so that it is not necessary to reconfigure clients, etc. with three IP addresses for the same queue manager. In this case, you will assign the floating address 10.0.0.10 to virtual adapter ens33, where currently you already have a fixed IP address configured on each virtual machine.

1. Stop (with **ctrl-C**) the HA sample programs that are currently running on each of the nodes.	***Do not close the terminal windows just yet as you will be running these programs again!***

1. Switch to **miqms**. As this is currently the primary node for the myRDQM queue manager, add the floating IP address, by running the command (as **ibmdemo**):

	```
	rdqmint –m myRDQM –a –f 10.0.0.10 -l ens34
	```
	![](./images/pots/mq-ha/lab2/image35.png)
	
1.	It is recommended to add another listener specifically for this floating IP address. By default, the standard listener will listen on the same port on every IP address, so a different port needs to be chosen for the additional listener. In a terminal window, create a listener by entering the runmqsc commands as follows: 

	```
	runmqsc myRDQM
	```
	
	```
	DEFINE LISTENER(FLOATING.LISTENER) TRPTYPE(TCP) CONTROL(QMGR) IPADDR(10.0.0.10) PORT(1501)
	```
	
	```
	end
	```
	
	![](./images/pots/mq-ha/lab2/image36.png)

1. Switch to **miqmp**. The new listener will not be started until the queue manager is restarted, so move the queue manager back to its original node by running the following command on node **miqmp** in a terminal window, as user ibmdemo:

	```
	rdqmadm –m myRDQM –p
	```
	![](./images/pots/mq-ha/lab2/image37.png)
		You can check that both listeners are running by running the following command:
	
	```
	netstat –ant | grep 150
	```

	![](./images/pots/mq-ha/lab2/image38.png)
	
	The first listener is the one created because -p **1500** was specified on the *crtmqm* command. This listener is listening on port 1500 on every IP address. The second listener, however, is listening on port **1501** on the floating IP address only.
	
1. Now that you have a floating IP address associated with myRDQM, you can change the MQSERVER environment variable to CHANNEL1/TCP/10.0.0.10(1501).	Locate the window where the amqsghac program was running and enter:	```
	export MQSERVER='CHANNEL1/TCP/10.0.0.10(1501)'
	```1. Now re-run the command: **amqsghac TARGET myRDQM**.

	```
	./amqsghac TARGET myRDQM
	```
	
	![](./images/pots/mq-ha/lab2/image39.png)
	
	**Leave this command to run!**
	
1. Switch to **miqms**. Locate the window where the amqsmhac program was running and enter:

	```
	export MQSERVER='CHANNEL1/TCP/10.0.0.10(1501)'
	```	1. Now re-run the command: **amqsmhac -s SOURCE -t TARGET -m myRDQM**

	```
	./amqsmhac -s SOURCE -t TARGET -m myRDQM
	```
	
	![](./images/pots/mq-ha/lab2/image40.png)
		**Leave this command to run!**

1. Switch to **mqnfs4**. Locate the window where the amqsphac program was running and enter:

	```
	export MQSERVER='CHANNEL1/TCP/10.0.0.10(1501)'
	```	1. Now re-run the command: **amqsphac SOURCE myRDQM**

	```
	./amqsphac SOURCE myRDQM
	```
	
	![](./images/pots/mq-ha/lab2/image41.png)
		**Leave this command to run!**

1. Now repeat the test. This time move it to mqnfs4. 

	a. On **mqnfs4**, in the other terminal window (where root is signed in), switch user to ibmdemo.  
	
	b. Issue the command to move the RDQM to **mqnfs4**. 
	
	![](./images/pots/mq-ha/lab2/image42a.png)
	
	c. Check the status of the queue manager to make sure it is running on **mqnfs4**. 
	
	![](./images/pots/mq-ha/lab2/image43.png)
	
	c. Confirm that messages continue to be sent and received without loss after the queue manager has been moved.	![](./images/pots/mq-ha/lab2/image44.png)
	
	![](./images/pots/mq-ha/lab2/image45.png)

1. When completed testing, move the RDQM to node 1 (**miqmp**), stop (with ctrl-C) the HA sample programs that are currently running on each of the nodes.

	
## CONGRATULATIONS! ### You have completed this hands-on lab.You have created replicated data queue managers to provide high availability for IBM MQ, and you have tested failing over. 