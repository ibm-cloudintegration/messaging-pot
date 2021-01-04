---
title: Multi-instance Queue Managers for High Availability 
toc: false
sidebar: labs_sidebar
folder: pots/mq-ha
permalink: /mq_ha_pot_lab1.html
summary: Multi-instance Queue Managers for High Availability
applies_to: administrator
---


# Software High Availability

![](./images/pots/mq-ha/lab1/image1.png)

## Multi-Instance Queue Managers

Author(s): Jack Carnes

Contributed by Brian Cuttell

**Acknowledgement**

The World Wide Team would like to thank Brian Cuttell of the BetaWorks Team for contributing this lab.

## Table of Contents

Part 1: Lab Introduction 

Part 2: Starting the images 

Part 3: Setting up the Network File System Server 

Part 4: Setting Up Network File System Clients 

Part 5: Setting Up Client Reconnect 

Part 6: MQ Explorer 

Part 7: More Failover Tests 

Part 8: Summary

## Lab Introduction

### Software High Availability Lab Overview

This lab will show how the software high availability feature of WebSphere MQ V9.1.5 can be set up in a Linux x86 environment. Similar steps must be followed in other Unix derived environments. 

### Lab environment

1. 2 RHEL 7.7 x86_64 systems running in Skytap: 

	* dr1  - NFS client primary node	* dr2  - NFS client secondary node
	* dr3  - NFS server

	{% include note.html content="There are three additional VMs in the Skytap template which are not used; rdqm1, rdqm2, and rdqm3 should remain suspended or powered off." %}
	
1. Network interfaces:

	|Interface Purpose | Interface Name |  dr1 (Primary node)  | dr2 (Secondary node) | dr3 (NFS server) |
	|:------:|:--------:|:--------:|:-----:|:--------:|:-------|
	| Administration | ens34 | 10.0.0.14 |10.0.0.15 | 10.0.0.16 |
	
The machines will be running on a Skytap network with the subnet 10.0.0.0. *dr3* is the host (server) for the Network File system. This will not have an MQ Queue manager running but will have the MQ Client installed. The server could be any system capable of exporting an NFSv4 file system.

The *dr1* and *dr2* images are machines which will have MQ installed – they will each have MQ 9.1.5 installed and will have the same queue manager defined operating as an active / passive pair. These two systems must be running the same operating system.

### Names used in this lab

| **Host Name** | **IP**          | **Purpose**                                                          |
|---------------|-----------------|----------------------------------------------------------------------|
| dr3        | 10.0.0.16| Host for NSFv4 system                                                 |
| dr1       | 10.0.0.14 | “Primary” host for queue manager. This is just a matter of naming.   |
| dr2        | 10.0.0.15 | “Secondary” host for queue manager. This is just a matter of naming. |

**Userids used in this lab**

| **Name**           | **UID / GID (These are arbitrary but must be same on all machines)** | **Purpose**                                |
|--------------------|----------------------------------------------------------------------|--------------------------------------------|
| Userid - “mqm”     | 1003                                                                 | The MQ userid – queue manager runs as mqm. |
| Groupid - “mqm”    | 1003                                                                | Primary group for mqm.                     |
| Userid - “ibmuser” | 1001                                                                 | A userid for logging on to console.        |

**NFS4 file system**

| **Name**            |                                                                                  |
|---------------------|----------------------------------------------------------------------------------|
| /NFS4FileSystem     | This is the root of the file system that is being exported.                      |
| /mnt/NFS4Filesystem | This is the mount point of the file system that is mounted by mq701p and mq701s. |
|                     |                                                                                  |

#### Logical Architecture

![](./images/pots/mq-ha/lab1/image2.png)

![](./images/pots/mq-ha/lab1/image3.png)

### Starting the images

In the Skytap environment, there are 6 virtual machines rdqm1, rdqm2, rdqm3, dr1, dr2, and dr3 which currently should be in a powered off or paused state.

![](./images/pots/mq-ha/lab1/image36.png)

This template is used for multiple labs and has been configured with the maximum number of VMs that are required for all labs. In this lab you will only need dr1, dr2 and dr3. The rest of the VMs can remain powered off or suspended.
1. Leave the labels checked for *dr1*, *dr2*, and *dr3*. Uncheck the labels for all other VMs. Then click the *run* button to start or resume the VMs.

	![](./images/pots/mq-ha/lab1/image37.png) 
	
	Wait for the monitor icons to turn green, approximately three minutes. Once they are green and running you can proceed to the next step.
	
2. Click the monitor icon for *dr3* which will launch the desktop in another browser tab.

	![](./images/pots/mq-ha/lab1/image38.png)

1. Log on to VM *dr3* as user **ibmuser**, using password **engageibm**.

	![](./images/pots/mq-ha/lab1/image39.png)
	
	![](./images/pots/mq-ha/lab1/image40.png)

### Configure NFS server 

The first step is setting up the Network File System. This must be created as an NFSv4 system and must be made available to the two other systems.We wish the MQ systems on the other two computers to be able to have unfettered read /write access to (at least part of) the file system. In order to allow this we will need to create “mqm” users and groups that have the same UID and GID numbers on each computer. The users and groups have been pre-configured on this environment.

1. You should still be logged onto *dr3*. Right click on the desktop and select *Open terminal*. 

	![](./images/pots/mq-ha/lab1/image41.png)
 	
1. Stop the firewall for initial testing. You can restart firewall later to test with firewall running. Use the following command to stop firewalld.

	```
	sudo systemctl stop firewalld
	```
	
	No response is given.
	
	![](./images/pots/mq-ha/lab1/image43.png)
	
1. Make a directory in which to store MQ logs and data to be shared with the NFS client machines. Enter the following commands:

	```
	sudo mkdir /NFS4FileSystem
	sudo mkdir /NFS4FileSystem/mq915
	sudo mkdir /NFS4FileSystem/mq915/log
	sudo mkdir /NFS4FileSystem/mq915/data
	```
	
	![](./images/pots/mq-ha/lab1/image43a.png)	
	
1. Change ownership of *NFS4FileSystem with following commands:

	```
	sudo chown -R mqm:mqm /NFS4FileSystem
	sudo chmod 775 /NFS4FileSystem
	```
	
	![](./images/pots/mq-ha/lab1/image43b.png)
	
1. 	You need to edit two system files. The first is */etc/exports*. You need root permissions to edit these files. Enter the following command to open the editor:

	```
	sudo gedit /etc/exports
	```
	
	Add the following line to the file then click *Save*.

	```
	/NFS4FileSystem *(rw,sync,no_wdelay,fsid=0)
	```
	
	![](./images/pots/mq-ha/lab1/image46.png)
 	
1. Now edit the file */etc/sysconfig/nfs*. Still in *gedit* click *Open*. Type */etc/sysconfig/nfs* and select **nfs**. 

	![](./images/pots/mq-ha/lab1/image47.png)
 	
1. Scroll down to the parameter *RPCMOUNTDOPTS* and add **-p 829** between the quotation marks. Click *Save*.

	![](./images/pots/mq-ha/lab1/image42.png)
	
1. After editing the file, you need to restart two services, nfs-config and nfs-server. Open another terminal window and run the following commands:

	```
	sudo systemctl restart nfs-config
	sudo systemctl restart nfs-server
	```
	
	![](./images/pots/mq-ha/lab1/image44.png)	
1. Now you can export the shared file system with the following command.

	```
	sudo exportfs
	```
	
	The response shows that the export was successful and is ready to be accessed by NFS clients.
	
	![](./images/pots/mq-ha/lab1/image45.png) 
	
### Configure NFS clients	Now each machine that is to access the NFS4 file system must be configured. We wish the MQ systems on the other two computers to be able to have unfettered read/write access to (at least part of) the file system. In order to allow this we will need to create “mqm” users and groups that have the same UID and GID numbers on each computer. As stated previously, these were pre-configured on this environment so there is nothing to do.

1. Start the *dr1* desktop and open a terminal window. 

1. On the NFS clients you need to add the shared file system to */etc/fstab*. Root access is required for this file also so open the editor with the following command: 

	```
	sudo gedit /etc/fstab
	```
	
	Add the following line at the end of the file:
	
	```
	dr3:/NFS4FileSystem /mnt/NFS4FileSystem nfs options 0 0
	```
	
	Click *Save*.
	
	![](./images/pots/mq-ha/lab1/image48.png)
	

1. You are now ready to mount the shared file system (on *dr1*). Open another terminal window to enter commands. First you must create a directory for mounting the shared file system. Enter the following command to create the directory.

	```
	sudo mkdir /mnt/NFS4FileSystem
	```
	
	Now you can mount the shared file system with the following command:

	```
	sudo mount -t nfs dr3:/NFS4FileSystem /mnt/NFS4FileSystem
	```	
	
	This command mounts the directory NFS4FileSystem on *dr3* (NFS server) onto the local mount point /mnt/NFS4FileSystem on *dr1* (NFS client).
	
	List the contents of the new directory and you will see the contents that you created on *dr3*. You should see the subdirecotry *mq915*.
	
	
	```
	ls -l /mnt/NFS4FileSystem
	```
	
	![](./images/pots/mq-ha/lab1/image51.png)	
1. IBM MQ provides a test program to verify the file system is configured properly. **amqmfsck** checks whether a shared file system meets the requirements for storing the queue manager data of a multi-instance queue manager. It tests that a file system correctly handles concurrent writes to a file and the waiting for and releasing of locks. 

	Issue the following command to test your mounted shared file system:
	
	```
	/opt/mqm/bin/amqmfsck -v /mnt/NFS4FileSystem
	```
	
	![](./images/pots/mq-ha/lab1/image52.png)
	
	Running the command with the *-v* (verbose) parameter shows the checks being made. You should receive a message that the checks were succuessful.
	
1. Repeat this procedure on the other NFS client *dr2* starting at *Configure NFS clients*.

	![](./images/pots/mq-ha/lab1/image53.png)
	
### Create and start active instance of the queue manager 

The following steps should be carried out on one of the MQ queue manager machines (NFS clients). It does not matter which. But for this lab, we will start on *dr1*. 
In this exercise on the NFS client machine *dr1*, you will:
* Create the queue manager called **QMMI** using the *-ld* and *-md* options to specify the location of the log data and the message data in the shared file system* Start the queue manager called **QMMI** using the *–x* option to indicate that this startup allows a standby instance
* Create queue manager **QMMI** specifying the path for the logs and queue manager data.

1. Create queue manager **QMMI** with the following command from the terminal as userid *ibmuser*: 

	```
	crtmqm -ld /mnt/NFS4FileSystem/mq915/log/ -md /mnt/NFS4FileSystem/mq915/data QMMI
	```
	
	![](./images/pots/mq-ha/lab1/image54.png)
	
1. Start the queue manager **QMMI** as a multi-instance queue manager with the following command: 

	```
	strmqm -x QMMI
	```
	
	The *-x* paramneter indicates that this startup allows a standby instance.			
	![](./images/pots/mq-ha/lab1/image55.png)

#### Define MQ ObjectsFor testing you need to define the queue manager objects.

* Define security
* Define a channel of type svrconn.
* Define a listener for port 1414.
* Start the listener.

1. Still on *dr1*, start the runmqsc command and enter the following commands:

	```
	runmqsc QMMI
	```
	
	Run the following MQSC commands (remember that MQ objects are case sensitive):
	
	```
	ALTER QMGR CHLAUTH(DISABLED) CONNAUTH(' ')
	```
	```
	REFRESH SECURITY TYPE(CONNAUTH)
	```
	```
	DEFINE QLOCAL(Q1) DEFPSIST(NO) REPLACE
	```	
	```
	DEFINE CHANNEL('SYSTEM.ADMIN.SVRCONN') CHLTYPE(SVRCONN) REPLACE 
	```
	```
	DEFINE LISTENER('A.LISTENER') TRPTYPE(TCP) CONTROL(QMGR) PORT(1414) REPLACE
	```
	``` 
	START LISTENER('A.LISTENER') 	
	```
	```
	END
	```
	
	![](./images/pots/mq-ha/lab1/image56.png)	1. Display the queue manager status.

	```
	dspmq -x -o all
	```
	
	This command will: 
	
	* Display the queue manager with respect to standby and giving all output.
	* We observe that the queue manager shows the STANDBY permitted option. And that the dr1 instance is active.

	![](./images/pots/mq-ha/lab1/image57.png)	
1. The next command will create the “ini.” definitions for the second (standby) instance. Enter the following command: 

	```
	dspmqinf -o command QMMI
	```
	
	* Command outputs the ini definitions in a command form that can be run on the second machine.
	* The response from the command is the text of a command that can be run on anothermachine where MQ is installed to add information allowing it to find the shared file system.

	![](./images/pots/mq-ha/lab1/image59.png)
	
1. Copy the text from the output of the above command. In the next section you will paste this text into a terminal window in *dr2* to run the command.
	
#### Setup the standby instance
	
The following steps should be carried out on the other NFS client, *dr2*. The queue manager that we are going to set up is the same queue manager that was created and configured previously on *dr1*.
In this exercise you will update this instance with the location of the queue manager data.

1. Open a terminal window just as you did on the other machine logged in as *ibmuser / engageibm*. Paste the command copied from *dr1* and hit enter.


	```
	addmqinf -s QueueManager -v Name=QMMI -v Directory=QMMI -v Prefix=/var/mqm -v DataPath=/mnt/NFS4FileSystem/mq915/data/QMMI
	```
	
	![](./images/pots/mq-ha/lab1/image58.png)

1. Start the standby instance. The start command is exactly the same as was used to start the active instance. Enter the following command:

	```
	strmqm -x QMMI
	```
	
	![](./images/pots/mq-ha/lab1/image60.png)
	
	The –x option to indicate that this startup allows a standby instance. Observe that a standby instance is started.

1. Display the queue manager status. Enter the following command to display all instances of the queue manager:

	```
	dspmq -x -o all
	```
	
	This command:
	* Displays the status of the queue manager with respect to standby and giving all output.
	* Observe that there are now two instances of queue manager QMMI.
	* QMMI shows that it is ACTIVE on *dr1* and in STANDBY on *dr2*.

	![](./images/pots/mq-ha/lab1/image61.png)

#### Configuration Summary

In the previous exercises you have:

* Configured three Linux machines to use NFS4 file system; one as the NFS server and two other machines mounting the shared file system. 
* Installed IBM MQ V9.1.5 Server (pre-installed) on the two machines that have the shared file system mounted (NFS clients). 
* Configured  IBM MQ with a queue manager that is running on both machines in an active / standby mode as shown below:

![](./images/pots/mq-ha/lab1/image2.png)
### Setting up client reconnect

#### Accessing the Network File System

The next step is to connect a client using the client reconnect feature that can access either instance of the queue manager.

The IBM MQ client code has been pre-installed on the NFSv4 host computer (*dr3*). This client will be able to access the highly available queue manager. We will also install the MQ explorer on this computer to use explorer with the multiple instances of a queue manager.

#### Using the sample programs

Two sample programs that illustrate the client reconnect program are shipped with IBM MQ. They are *amqsphac* and *amqsghac* which show putting and getting respectively.

1. On *dr3*, open a terminal window and open a gedit session to examine the sample program *amqsphac*.

	```
	gedit /opt/mqm/samp/amqsphac.c
	```
	
	![](./images/pots/mq-ha/lab1/image62.png)	
1. Click the hamburger menu in the top right corner and select *Find*. 

	![](./images/pots/mq-ha/lab1/image63.png)
	
1. In the search field type in **cno.Options** to find the reconnection code. Examining the source code of the program (fragment shown below) shows the use of the new MQCONNX options.

	![](./images/pots/mq-ha/lab1/image64.png)

1. Do another find for **EventHandler** to examine the callback function to be notified of reconnect events.

	![](./images/pots/mq-ha/lab1/image65.png)
	
1.  If you wish, continue to review the full program before closing the edit session.

1. You will need two command windows; one for the get application and one for the put application. Open a terminal window logged in as *ibmuser / engageibm*. If you are still root, just enter *exit*.

1. Set the environment variable *MQSERVER* with the following command:

	```
	export MQSERVER=SYSTEM.ADMIN.SVRCONN/TCP/10.0.0.14,10.0.0.15
	```
	
	The client uses this variable to find instances of the queue manager. Observe that we now supply a list of ip addresses for the connection separated by commas. Notice that there are no spaces.

1.  In this command window start putting messages by starting the sample programn *amqsphac*.

	```
	/opt/mqm/samp/bin/amqsphac Q1 QMMI
	```
	
	![](./images/pots/mq-ha/lab1/image68.png) 
	
	Notice that the sample putting program begins to put messages (about one per second) to Q1 on qmgr QMMI. Leave this command window open to run.

1.  Open another terminal and position it so that you can see both terminal windows.

1.  Set the environment variable *MQSERVER* with the following command:

	```
	export MQSERVER=SYSTEM.ADMIN.SVRCONN/TCP/10.0.0.14,10.0.0.15
	```
	
	The client uses this variable to find instances of the queue manager. Observe that we now supply a list of ip addresses for the connection separated by commas. Notice that there is no spaces.

1.  In this command window start getting messages by starting the sample program *amqsghac*.

	```
	/opt/mqm/samp/bin/amqsghac Q1 QMMI
	```
	
	![](./images/pots/mq-ha/lab1/image69.png)
	
	Notice that the sample getting program gets all of the messages from *Q1* on qmgr *QMMI* that have been put to date and then keeps pace with the putting program, displaying each message as it arrives. 
	
	Leave this command window open to run.

#### Failing over the queue manager

While these two programs are running we will failover the queue manager and observe the results.

1.  Verify that the two windows are still open on *dr3* and that the sample programs are still running, putting and getting messages.

2.  Go to the machine where the active instance of the queue manager **QMMI** is running, *dr1* or *dr2*. For this lab it should be *dr1*.

1.  Ensure that you are logged in as userid *ibmuser / engageibm*.

2.  In a terminal session, stop the queue manager with the *-i -s* option to shutdown the queue manager allowing for failover. Enter the following command:

	```
	endmqm -i -s QMMI
	```
	
1.  After the queue manager has shut down, restart it as a standby instance using the *strmqm* command with the *–x* option.
	
	```
	strmqm –x QMMI
	```
	
	![](./images/pots/mq-ha/lab1/image70.png)	

1. Switch back to the machine where the MQ client code is running (*dr3* in this case). You will see something similar to the screen shots below for both command windows where the sample programs are running.

	![](./images/pots/mq-ha/lab1/image71.png)


	Messages are put until the switchover. At which point the event listener code reports the connection status. After reconnection is established, the putting program goes on as before but this time the queue manager is running on the other instance.

	NOTES: The putting program in this illustration displays all the records being put. However it is possible that some messages may be lost, because the messages are being put out of sync-point as non persistent massages.

1.  Verify where the queue manager is running by displaying MQ on both *dr1* and *dr2*. Enter the following command on both *dr1* and *dr2*:

	```
	dspmq -m QMMI
	```
	
	As shown below, you should see that QMMI is no longer running on *dr1*. But now it is running on *dr2* where it failed over. The standby has become active and the old active is now on standby.

	![](./images/pots/mq-ha/lab1/image72.png)
	
	![](./images/pots/mq-ha/lab1/image73.png)
	
#### Basic Failover Summary

During this failover demonstration we have:

* Configured an MQ client connection that identifies two instances of a queue manager as connection targets.

* Observed the changes required in an MQ API program to support client reconnect.

* Seen a failover occur and the client connected application keep running while the standby queue manager took over.

* The configuration is as shown below

![](./images/pots/mq-ha/lab1/image74.png)

### MQ Explorer

The MQ client code and the MQ Explorer were pre-installed on the NFS server - *dr3s*. In order to demonstrate multi-instance queue managers on MQ Explorer, we must return to *dr3* and start the MQ Explorer eclipse workbench.

1. On the NFS server machine *dr3* in a terminal logged in as *ibmuser*, enter the following command:

	```
	MQExplorer
	```
	
	This will launch the eclipse based *MQ Explorer*.
	
	![](./images/pots/mq-ha/lab1/image75.png)
	
1.  In the left hand navigation bar, right-click on *Queue Managers* and select *Add Remote Queue Manager*.

	![](./images/pots/mq-ha/lab1/image76.png)	
1.  Enter the queue manager name as **QMMI**. Select *Connect directly* to an active instance. Click the *Next* button

    ![](./images/pots/mq-ha/lab1/image77.png)

4.  Enter the connection details, either the IP address 10.0.0.15 or host name *dr2*. Leave the port and channel values at the default.  Mark the checkbox for *Is this a multi-instance queue manager?* 

	Enter the connection details for the second instance, either the IP address 10.0.0.14 or host name *dr1*. Leave the port and channel values at the default. Then click *Next*.
	
	![](./images/pots/mq-ha/lab1/image78.png)
	
1. Click *Next* at the "Specify security exit details" screen.

1. On the "Specify user identification details" screen check the *Enable user identification* checkbox. Enter *ibmuser* in the Userid field. Check the *Prompt for password* radio button. Notice the error message that *Prompt for password* is not allowed for multi-instance queue managers. 

	![](./images/pots/mq-ha/lab1/image78a.png)

1. On the *Preferences* page check the radio button for *Save passwords to file* then click *Apply and Close*.

	![](./images/pots/mq-ha/lab1/image78b.png)
	
1. Now click the *Use saved password* radio button and click *Enter password*.

	![](./images/pots/mq-ha/lab1/image79.png)

7.  Enter the password **engageibm** and click *OK*. Click *Finish*.

    ![](./images/pots/mq-ha/lab1/image80.png)

1.  The queue manager will be added to those managed by the *MQ Explorer*.

	![](./images/pots/mq-ha/lab1/image81.png)

8.  Click on the icon to expand the *QMMI** queue manager. Click on *Queues* to see the local queue **Q1** added earlier in the lab.

	![](./images/pots/mq-ha/lab1/image82.png)

1.  Right-click on the *QMMI* queue manager. Select *Connection Details*, then *Manage Instances*.

	![](./images/pots/mq-ha/lab1/image83.png)

1.  Review *Manage Instances* window. Note that the *second instance* (10.0.0.15 – dr2) is the one that is connected. This should be expected since the queue manager failed over from *dr1* to *dr2* earlier in the lab.

	![](./images/pots/mq-ha/lab1/image84.png)

1.  Click *Test Connections*.
	
	Note that the first instance (10.0.0.14 – d1) shows *Not available*.
	Also note that you can reorder the list by clicking on *Move up* or *Move down*.
	
	![](./images/pots/mq-ha/lab1/image85.png)

1.  Click *Close* to end the dialog.

1.  Add a Local Queue by right-clicking *Queues*. Select *New* > *Local Queue*.

	![](./images/pots/mq-ha/lab1/image86.png)

1.  Enter the name **Q2**. This queue will be used in the next part of the lab. Press *Next*.

	![](./images/pots/mq-ha/lab1/image87.png)

1.  Click the the drop-down for *Default persistence* field and change to **Persistent**. Click *Finish*.

	![](./images/pots/mq-ha/lab1/image88.png)
	
1. Click *OK* to dismiss the pop-up confirming success.

### More Failover Tests

You have completed the lab, but you may want to work on this part if time permits. This section will give you more practice and further your understanding of fail-over and client reconnection.

#### Persistent and non-persistent messages

The test program *amqsphac* puts messages with the default persistence defined for the queue. Non-persistent messages DO NOT survive a failover between instances.

1. Return to the *dr3* machine that connects as a client and work as *ibmuser*.

2. Open four terminals and set the environment variable *MQSERVER* in each command window. 

	Note: You may still have your open terminals running the sample programs. In this case you will only need to start open two additional terminals. If the sample programs are still running, please stop them by hitting *ctrl-C*. Position the windows, so you can see all four. 
	
	```
	export MQSERVER=SYSTEM.ADMIN.SVRCONN/TCP/10.0.0.14,10.0.0.15
	```
	
1. In terminal 1, start the putting application with *non-persistent* messages by putting messages to *Q1*.

	```
	/opt/mqm/samp/bin/amqsphac Q1 QMMI
	```

3. In terminal 3 (new terminal), start the putting application with *persistent* messages by putting messages to *Q2*. 

	```
	/opt/mqm/samp/bin/amqsphac Q2 QMMI
	```
	
	![](./images/pots/mq-ha/lab1/image89.png)
	
5. On the machine that is running the active instance of the queue manager (should be *dr2*), make sure you are logged in as *ibmuser*. End the queue manager with the following command allowing switchover.

	```
	endmqm -i -s QMMI
	```
	
	Note: If you forgot to restart the queue manager as *standby* on *dr1*, you will get a message that says switchover is not possible. 

1. Restart the queue manager in standby mode with the following command: 

	```
	strmqm –x QMMI
	```

1. Switch back to *dr3*.

2. In terminal 2, start the getting application for *non-persistent* messages from *Q1*.

	```
	/opt/mqm/samp/bin/amqsghac Q1 QMMI
	```

4. In terminal 4, start the getting application for *persistent* messages from *Q2*.

	```
	/opt/mqm/samp/bin/amqsghac Q2 QMMI
	```

	*Terminal 1 – amqsphac – Q1 non-persistent* 
	*Terminal 2 – amqsghac – Q1 non-persistent* 

	![](./images/pots/mq-ha/lab1/image92.png)

	*Terminal 3 – amqsphac – Q2 persistent* 
	*Terminal 4 – amqsghac – Q2 persistent*

	![](./images/pots/mq-ha/lab1/image93.png)
	
1.  Failure of the Queue Manager Host - the failover tests can be repeated, but instead of stopping the queue manager with the endmqm *-i* *-s* option simply kill the machine that the queue manager is running on or disable its network adapter. Failover will occur. Give it a try!

## Summary

In this lab you have:

* Configured 3 LINUX machines to use NFSv4 file system, one as host with two machines mounting the shared file system.

* Configured IBM MQ Server on the two machines that have mounted the file system.

* Configured IBM MQ with a queue manager that is running on both machines in an active/standby configuration.

* Configured an MQ client connection that identifies two instances of a queue manager as connection targets.

* Observed the changes required in an MQ API program to support client reconnect.

* Seen a failover occur and the client connected application keep running while the standby queue manager took over.

* Seen MQ Explorer with the multiple instance data.


## CONGRATULATIONS! 

### You have completed this hands-on lab.

 
[Continue to Lab 2](mq_ha_pot_lab2.html)

[Return MQ HA Menu](mq_ha_pot_overview.html)