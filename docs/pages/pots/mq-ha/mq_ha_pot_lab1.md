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
![](./images/pots/mq-ha/lab1/miqm-image1.png)

<img src="media/image1.png" width="131" height="76" />

## Multi-Instance Queue Managers

Author(s): Jack Carnes

Contributed by Brian Cuttell

**Acknowledgement**

The World Wide Team would like to thank Brian Cuttell of the BetaWorks Team for contributing this lab.

Version 1.0

## Table of Contents

Part 1: Lab Introduction 3

Part 2: Starting the images 7

Part 3: Setting up the Network File System Server 19

Part 4: Setting Up Network File System Clients 26

Part 5: Setting Up Client Reconnect 41

Part 6: MQ Explorer 52

Part 7: More Failover Tests 61

Part 8: Summary 63

## Lab Introduction

### Software High Availability Lab Overview

This lab will show how the software high availability feature of WebSphere MQ V7.0.1 can be set up in a Linux X86 environment. Similar steps must be followed in other Unix derived environments.

### Environment

This lab was developed using 3 Linux machines running under VMWARE. The Linux distribution is Suse Linux Enterprise Server 10 SP1. By now, you should have received the VMware image archive files. If not, notify the instructor. The archive is self-extracting. Just double-click on the first file (\*.exe), specify the destination directory, and the images will be extracted. The archive contains all three images as they are defined as VMware clones.

The machines will be running on a VMWARE NAT network with the subnet 192.168.145.0. The first image is the host (server) for the Network File system. This will not have an MQ Queue manager running but will have the MQ Client installed. The server could be any system capable of exporting an nfsv4 file system.

The second and third images are machines which will have MQ installed – they will each have MQ 7.0.1 installed and will have the same queue manager defined operating as an active / passive pair. These two systems must be running the same operating system.

### Names used in this lab

| **Host Name** | **IP**          | **Purpose**                                                          |
|---------------|-----------------|----------------------------------------------------------------------|
| mqnsf4        | 192.168.145.111 | Host for NSF4 system                                                 |
| mq701p        | 192.168.145.112 | “Primary” host for queue manager. This is just a matter of naming.   |
| mq701s        | 192.168.145.113 | “Secondary” host for queue manager. This is just a matter of naming. |

**Userids used in this lab**

| **Name**           | **UID / GID (These are arbitrary but must be same on all machines)** | **Purpose**                                |
|--------------------|----------------------------------------------------------------------|--------------------------------------------|
| Userid - “mqm”     | 1111                                                                 | The MQ userid – queue manager runs as mqm. |
| Groupid - “mqm”    | 1111                                                                 | Primary group for mqm.                     |
| Userid - “student” | 1113                                                                 | A userid for logging on to console.        |

**NFS4 file system**

| **Name**            |                                                                                  |
|---------------------|----------------------------------------------------------------------------------|
| /NFS4FileSystem     | This is the root of the file system that is being exported.                      |
| /mnt/NFS4Filesystem | This is the mount point of the file system that is mounted by mq701p and mq701s. |
|                     |                                                                                  |

#### Logical Architecture

#### Multi-Instance Queue Manager

![](./images/pots/mq-ha/lab1/miqm-image2.png)

![](./images/pots/mq-ha/lab1/miqm-image3.png)


### Starting the images

By this time you should have completed the extraction of three VMware images. After extraction, you will find a directory structure on your destination disk that looks like the screen shot below.

![](./images/pots/mq-ha/lab1/miqm-image4.png)


***DO NOT START the Images at this time.***

***You must start the images in the EXACT order described in the following instructions.***

This section will lead you through the startup of each image.

You will find \*.vmx files for each image. We will refer to the images by their hostnames **mqnfs**, **mq701p**, **mq701s**. Notice that there is a directory for **mqnfs** and **mq701s**, but not **mq701p**. The **\*.vmx** file for **mq701p** is located in the root directory of the destination location when extracted. In this case, it is in the **ConnSTEW2010-Linux-Clones** directory.

1.  **Open Images in VMware**

    Start the VMware program on your desktop.

    Click on **File**, select **Open.**

> <img src="media/image5.png" width="527" height="456" />

Navigate to the directory where you extracted the images.

Select the **mq701p ConnHQqmgr.vmx** file and press **Open**.

> <img src="media/image6.png" width="586" height="402" />

Anytime during the start-up process, you may be asked if you copied or moved this image. Check **I copied it** and click **OK**.

> <img src="media/image7.png" width="469" height="232" />

VMware opens the image to the **Summary** page of the image as shown below. Notice that it displays the state of your virtual machine as well as the path to the location where you extracted this instance of the image.

You are now ready to power up the virtual machine. Click on **Power on this virtual machine.**

> <img src="media/image8.png" width="613" height="454" />

Click on **File** **Open**. You will now see additional “lock” directories. Navigate the mqnfs sub-directories until you find the **\*.vmx** file for **mqnfs**.

> <img src="media/image9.png" width="593" height="407" />

Select the **SLES10SP1-32.vmx** file and click **Open**.

> <img src="media/image10.png" width="596" height="414" />

VMware opens the image to the **Summary** page of **mqnfs** as shown below. Notice that it displays the state of your virtual machine as well as the path to the location where you extracted this instance of the image.

You are now ready to power up the virtual machine. Click on **Power on this virtual machine**.

> <img src="media/image11.png" width="607" height="443" />

Click on **File** **Open**. Navigate up until you find the **mq701s** sub-directory.

> <img src="media/image9.png" width="593" height="407" />

Double click the **mq701s** directory. You will find the **\*.vmx** file for **mq701s**.

Select the **mq701s ConnHAqmgr.vmx** file and click O**pen**.

> <img src="media/image12.png" width="600" height="410" />

VMware opens the image to the **Summary** page of **mq701s** as shown below. Notice that it displays the state of your virtual machine as well as the path to the location where you extracted this instance of the image.

> <img src="media/image13.png" width="603" height="441" />

At this point, all three images should be powered up or in the process of powering up. You will have three tabs in the VMware workspace, one for each of the images.

> <img src="media/image14.png" width="421" height="194" />

1.  **Log in the Virtual Machine**

> When each virtual machine has completed its power-on process, you will be presented with the sign-on panel. In order to change from one machine to another, just click on its tab in the VMware workspace. Start with the image name mqnfs ConnHAqmgr.

1.  Click inside **Username** and enter “virtuser” and press &lt;enter&gt;**.**

2.  Enter “**virtuser**” again in the **Password** field and press &lt;enter&gt;.

> <img src="media/image15.png" width="370" height="148" />

1.  When you are completely logged-in, you will have a clean desktop shown here.

> <img src="media/image16.png" width="570" height="426" />

1.  Repeat the sign-on process for the other virtual machines **mq701p**, **mq701s**.

<!-- -->

1.  **Using root access.**

    Throughout the rest of the lab, you may be asked to **switch user** to **root**.

<!-- -->

1.  To switch user to root use the **su –** command.

> <img src="media/image17.png" width="401" height="290" />

1.  Enter “**passw0rd**” for the password when prompted.

Setting up the Network File System Server
=========================================

**Setup on Host Computer**

The first step is setting up the Network File System. This must be created as an nfs4 system and must be made available to the two other systems.

We wish the MQ systems on the other two computers to be able to have unfettered read/write access to (at least part of) the file system. In order to allow this we will need to create “mqm” users and groups that have the same UID and GID numbers on each computer.

1.  **Creating Users on mqnfs**

<!-- -->

1.  In this exercise on the NFS Server machine **mqnfs**, you will create these users and groups:

<!-- -->

1.  Create user mqm UID 1111

2.  Create user student UID 1113

3.  Create group GID 1111

> The **student** userid is intended as a sign-on id and will need a password and home directory.

1.  The following commands should be entered after **switching to root** (remember the command to switch to root? **su -) **

<!-- -->

1.  **groupadd –g 1111 mqm**

    -   creates the mqm group with GID 1111g

2.  **useradd –u 1111 –g mqm mqm**

    -   creates the mqm user with UID 1111 and with group mqm as primary group

3.  **useradd –u 1113 –g users –G mqm –d /home/student student**

    -   creates the student user as a member of groups users and mqm with home directory /home/student

4.  **passwd student**

    -   Sets the password for the user student. You will be prompted for the password. In this lab I used “student” as the password. Note: the system complains about the simplicity of the password, but accepts it.

5.  **mkdir /home/student**

    -   Creates the home directory.

6.  **chown student:users /home/student**

    -   Makes the student user the owner of its home directory.

Check the screen shot below to compare your commands.

<img src="media/image18.png" width="401" height="292" />

1.  Click on **Computer**, select **Logout**.

> <img src="media/image19.png" width="519" height="282" />

1.  Click on **Logout** on the next pop-up window.

> <img src="media/image20.png" width="278" height="191" />

1.  Sign back into the NFS server host – mqnfs with userid “student” password “student”.

> <img src="media/image21.png" width="384" height="134" />

1.  Open a Gnome terminal by right-clicking on **Computer** and selecting **Gnome Terminal**.

> <img src="media/image22.png" width="521" height="309" />

1.  **Creating the NFS4 file system on mqnfs**

<!-- -->

1.  Still on the NFS Server machine **mqnfs4**, you will now create the file system to be used by all three machines. Tasks to be done are:

<!-- -->

1.  Create a directory to be the root of the NFS4 file system.

2.  Create subdirectories for the MQ data and logs.

3.  Change ownership to mqm.

<!-- -->

1.  The following commands should be used after **switching to root**.

<!-- -->

1.  **mkdir /NFS4FileSystem**

    -   creates root directory for the shared file system

2.  **mkdir /NFS4FileSystem/mq701**

> **mkdir /NFS4FileSystem/mq701/log**
>
> **mkdir /NFS4FileSystem/mq701/data**

-   creates the subdirectories for MQ to user

1.  **chown –R mqm:mqm /NFS4FileSystem**

    -   makes **mqm** user the owner of the nfs4 file system.

2.  **chmod –R 775 /NFS4FileSystem**

    -   Allows all users read and execute.

    -   Allows **mqm** user and group full access.

3.  **su mqm **

-   switch the user to

1.  **cat &gt; /NFS4FileSystem/TestFile.txt** &lt;enter&gt; **test data** &lt;enter&gt; &lt;ctrl-D&gt;

    -   Creates a test file on the shared file system.

Check the screen shot below to compare your commands.

> <img src="media/image23.png" width="400" height="288" />

1.  **Exporting the file system**

    The created file system must be made available to other systems. A line must be added to the /etc/exports file on the nfs4 server. The line should read:

    **/NFS4FileSystem \*(fsid=0,rw,sync,no\_wdelay,no\_subtree\_check,insecure)**

<!-- -->

1.  You should still have a Gnome terminal open on the NFS Server machine mqnfs and su’ed to root. Note: Remember to exit userid mqm if you are still su’ed to mqm. You then should be su’ed to root.

2.  The following command(s) should be used after **switching to root**. (if not already root)

<!-- -->

1.  **vi /etc/exports**

    -   vi is the command line editor. You may use the File Manger GUI interface if you wish.

> <img src="media/image24.png" width="401" height="161" />
>
> Press the &lt;insert&gt; key. The cursor will be positioned on a blank line. Enter the following line:
>
> **/NFS4FileSystem \*(fsid=0,rw,sync,no\_wdelay,no\_subtree\_check,insecure)**
>
> Note: There is no space after the “\*”.
>
> <img src="media/image25.png" width="401" height="291" />

1.  Save the file with the following command string:

> &lt;esc&gt; **:wq **
>
> <img src="media/image26.png" width="398" height="285" />

1.  **/etc/init.d/idmapd start **

    -   Start he userid mapping demon.

2.  **/etc/init.d/nfsserver start**

    -   Start (or could user “restart”) the NFS server.

3.  **exportfs**

    -   Export the file system.

4.  **showmount –exports**

    -   Display the exported file system. Please make sure to enter two hyphens before “exports” in this command. One hyphen will not work.

5.  **Compare your commands to the screen shot below.**

<img src="media/image27.png" width="397" height="146" />

Setting Up Network File System Clients
======================================

**Accessing the Network File System**

Now each machine that is to access the NFS4 file system must be configured.

We wish the MQ systems on the other two computers to be able to have unfettered read/write access to (at least part of) the file system. In order to allow this we will need to create “mqm” users and groups that have the same UID and GID numbers on each computer.

1.  **Creating Users on mq701p and mq701s**

<!-- -->

1.  User creation is just the same as on the NFS server. You should still have a Gnome terminal open on the NFS client machine mq701x (**p** or **s**) and su’ed to root.

2.  The following command(s) should be entered after **switching to root**: (you may already be su’ed to root)

<!-- -->

1.  **groupadd –g 1111 mqm**

    -   creates the mqm group with GID 1111g

2.  **useradd –u 1111 –g mqm mqm**

    -   creates the mqm user with UID 1111 and with group mqm as primary group

3.  **useradd –u 1113 –g users –G mqm –d /home/student student**

    -   creates the student user as a member of groups, users and mqm, with home directory /home/student

4.  **passwd student**

    -   Sets the password for the “student” user. You will be prompted for the password. In this lab I used “student” as the password. Note: the system complains about the simplicity of the password, but accepts it.

5.  **mkdir /home/student**

    -   Creates the home directory.

6.  **chown student:users /home/student**

    -   Makes the student user the owner of its home directory.

> Check the screen shot below to compare your commands.
>
> <img src="media/image28.png" width="511" height="264" />

1.  Click on **Computer**, select **Logout**.

<img src="media/image19.png" width="519" height="282" />

1.  Click on **Logout** on the next pop-up window.

> <img src="media/image20.png" width="278" height="191" />

1.  Open a Gnome terminal by right-clicking on **Computer** and selecting **Gnome Terminal**.

> <img src="media/image22.png" width="521" height="309" />

1.  **Mounting the NFS4 File System**

<!-- -->

1.  The following command(s) should be used after **switching to root**. (if not already root)

<!-- -->

1.  **mkdir /mnt/NFS4FileSystem**

    -   creates the mount point for the shared file system

2.  **mount –t nfs4 192.168.xxx.111:/ /mnt/NFS4FileSystem**

    -   make sure to replace xxx with your subnet.

    -   mounts the file system at the appropriate point.

    -   Note: the NFS host is followed by “**:/**”, a space, then the mount point.

3.  **/etc/init.d/idmapd start**

    -   Start the userid mapping daemon on this computer – **mq701x** (where x is **p** or **s**)

> <img src="media/image29.png" width="398" height="292" />

***IMPORTANT ***

1.  It is important to restart the id-mapping daemon on the nfs server to pick up the new userid defined on the NFS client machine. You must restart it each time you update a nfs client.

> Switch back to the NFS server machine – mqnfs and restart the idmapd daemon from the Gnome Terminal with root access using the following command:
>
> **/etc/init.d/idmapd restart**
>
> <img src="media/image30.png" width="544" height="394" />

1.  Now, back on the NFS client machine – **mq701x** (where **x** is p or **s**), enter the following commands:

<!-- -->

1.  **exit**

    -   Logs out of root user, returns to userid “student”

2.  **ls –l /mnt/NFS4FileSystem/mq701**

    -   Display the shared file system. This command should report that the **data** and **log** directories as being owned by mqm.

<img src="media/image31.png" width="521" height="375" />

1.  **Repeat Part 4, Steps 1 – 2.a.2 for the other NFS client machine mq701s.**

2.  **Installing WMQ V7.0.1 on the NFS clients**

    The two machines that are to make up an active / standby pair must have MQ installed. The WebSphere MQ V7.0.1 for Linux rpm files for installing WMQ server are located in the **/STEW/wmqv701** directory.

    The following steps should be carried out on both machines.

<!-- -->

1.  On the NFS client machines **mq701p** and **mq701s**, you will

    -   Accept the license agreement by executing the mqlicense command

    -   Perform the installation using the rpm command.

2.  Install WMQ server with the following commands from the Gnome Terminal:

<!-- -->

1.  **su – **

    -   switch user to root, enter password “passw0rd”

2.  **cd /STEW/wmqv701**

    -   change to directory where the rpm files have been untarred

3.  **./mqlicense.sh**

    -   Run the license command

> <img src="media/image32.png" width="528" height="380" />

-   Accept license agreement

> <img src="media/image33.png" width="528" height="383" />

1.  **rpm –ivh MQSeriesRuntime-7.0.1-0.i386.rpm**

2.  **rpm –ivh MQSeriesServer-7.0.1-0.i386.rpm**

3.  **rpm –ivh MQSeriesJRE-7.0.1-0.i386.rpm**

    -   Install the required code for WMQ server

    -   Compare your commands to the screenshot below.

> <img src="media/image34.png" width="518" height="374" />

1.  Verify that WMQ is able to use the shared file system by entering the following commands:

<!-- -->

1.  **Exit**

    -   exit user root, switching back to userid student

2.  **cd /opt/mqm/bin/ **

    -   change to the mqm bin directory where the executables are

3.  **./amqmfsck /mnt/NFS4FileSystem/mq701**

    -   Run new sample program “amqmfsck” to make sure that the shared file system can be locked

> <img src="media/image35.png" width="549" height="95" />

1.  Repeat Step 4a – 4c3 above for the other NFS (**mq701s**) client machine.

> **
> **

1.  **Create and Start Active Instance**

    The following steps should be carried out on one of the MQ queue manager machines. It does not matter which. But for this lab, we will start on mq701p.

<!-- -->

1.  In this exercise on the NFS client machine **mq701p**, you will

-   Create the queue manager called MQ701 using the –ld and –md options to specify the location of the log data and the message data in the shared file system.

-   Start the queue manager called MQ701 using the –x option to indicate that this startup allows a standby instance.

> Create and start queue manager MQ701 with the following commands from the Gnome Terminal as userid “student”:

1.  **crtmqm –ld /mnt/NFS4FileSystem/mq701/log/ -md /mnt/NFS4FileSystem/mq701/data/ MQ701**

    -   create queue manager **MQ701** specifying the path for the logs and queue manager data.

> <img src="media/image36.png" width="535" height="389" />

1.  **strmqm –x MQ701**

    -   start queue manager **MQ701** using the –x option to indicate that this startup allows a standby instance.

> <img src="media/image37.png" width="541" height="121" />

1.  **Define MQ Objects**

    The following commands should be used after switching to userid “student”. (You should already be logged in as “student”)

<!-- -->

1.  Start the runmqsc program for entering MQ commands.

<!-- -->

1.  **runmqsc MQ701**

<!-- -->

1.  Define SVRCONN Channel and Listener with the following commands:

<!-- -->

1.  **DEFINE CHANNEL(SYSTEM.ADMIN.SVRCONN) CHLTYPE(SVRCONN)**

    -   Define a channel of type svrconn.

2.  **DEFINE LISTENER(A.LISTENER) TRPTYPE(TCP) CONTROL(QMGR) PORT(1414)**

    -   Define a listener for port 1414.

3.  **START LISTENER(A.LISTENER)**

    -   Start the listener.

<!-- -->

1.  Compare your commands to the screen shot below.

<img src="media/image38.png" width="530" height="385" />

1.  Define a Local Queue.

    You should still be logged in as “student” and runmqsc should still be running in a Gnome terminal. Enter the following commands:

<!-- -->

1.  **define qlocal(q1)**

    -   Define a local queue called **Q1**.

2.  **end**

    -   Stop runmqsc.

> <img src="media/image39.png" width="535" height="148" />

1.  Display the queue manager status.

    You should still be logged in as “student” in a Gnome terminal. Enter the following command:

<!-- -->

1.  **dspmq –x –o all**

    -   display the queue manager with respect to standby and giving all output.

    -   We observe that the queue manager shows the STANDBY permitted option. And that the mq701p instance is active.

> <img src="media/image40.png" width="549" height="96" />

1.  Create the “ini.” definitions for the second instance.

    You should still be logged in as “student” in a Gnome terminal. Enter the following command:

<!-- -->

1.  **dspmqinf –o command MQ701**

    -   Output the ini definitions in a command form that can be run on the second machine.

    <!-- -->

    -   The response from the command is the text of a command that can be run on another machine where MQ is installed to add information allowing it to find the shared file system.

> <img src="media/image41.png" width="536" height="64" />

-   Copy the output text to the clipboard by dragging your mouse to highlight the text, click on Edit and select Copy.

> <img src="media/image42.png" width="397" height="245" />

**
**

1.  **Set up the STANDBY Instance**

    The following steps should be carried out on the other NFS client, **mq701s**. The queue manager that we are going to set up is the same queue manager that was created and configured previously on **mq701p**.

<!-- -->

1.  In this exercise you will

    -   Update this instance with the location of the queue manager data

        Open a Gnome Terminal just as you did on the other machine. The following commands should be used after switching to userid “student”: (you should already be logged in as “student”)

        At the command prompt, you can now paste the command from the clipboard that you copied in the previous step. Click on Edit, select Paste from the pull-down menu, and hit &lt;enter&gt;.

        <img src="media/image43.png" width="400" height="292" />

        The text show match the line below.

<!-- -->

1.  **addmqinf –s QueueManager –v Name=MQ701 –v Directory=mq701 –v Prefix /var/mqm –v DataPath=/mnt/NFS4FileSystem/mq701/data/MQ701 **

    -   Use the **addmqinf** command. The input parameters used here are the ones output from the **dspmqinf** command on **mq701p**.

> <img src="media/image44.png" width="549" height="67" />

1.  Start the **standby instance.**

    The start command is exactly the same as was used to start the active instance. Enter the following commands:

<!-- -->

1.  **strmqm –x MQ701**

    -   Start the queue manager called **MQ701** using the –x option to indicate that this startup allows a standby instance.

    -   Observe that a **standby** instance is started.

> <img src="media/image45.png" width="457" height="327" />

1.  Display the queue manager status

    Enter the following command to display all instances of the queue manager:

<!-- -->

1.  **dspmq –x –o all**

    -   Display the status of the queue manager with respect to standby and giving all output.

    -   Observe that there are now two instances of queue manager MQ701.

    -   MQ701 shows that it is **Active** on **mq701p** and in **STANDBY** on **mq701s**.

> <img src="media/image46.png" width="500" height="139" />
>
> **
> **

1.  **Installation and Configuration Summary**

> In the previous exercises we have:

-   Configured three Linux machines to use NFS4 file system; one as the NFS server and two other machines mounting the shared file system.

-   Installed WebSphere MQ V7.0.1 Server on the two machines that have the shared file system mounted (NFS clients).

-   Configured WebSphere MQ with a queue manager that is running on both machines in an **active / standby** mode as shown below:

> <img src="media/image2.png" width="624" height="348" />

Setting Up Client Reconnect
===========================

**Accessing the Network File System**

> The next step is to connect a client using the client reconnect feature that can access either instance of the queue manager.
>
> The WebSphere MQ client code will be installed on the NFS4 host computer. This client will be able to access the highly available queue manager. We will also install the MQ explorer on this computer to use explorer with the multiple instances of a queue manager.

1.  **Install the WMQ client code**

<!-- -->

1.  On the NFS server machine **mqnfs**

-   Accept the license agreement by executing the mqlicense command

-   Perform the client installation using the rpm command.

-   Packages are shown in the screen shot – Runtime, Client, JRE, Eclipse, Samples, Config.

Install WMQ client with the following commands from the Gnome Terminal:

1.  **su – **

    -   switch user to root, enter password “passw0rd”

2.  **cd /STEW/wmqv701**

    -   change to directory where the rpm files have been untarred

3.  **./mqlicense.sh**

    -   Run the license command

> <img src="media/image47.png" width="489" height="355" />

-   Accept license agreement

> <img src="media/image33.png" width="528" height="383" />

1.  **rpm –ivh MQSeriesRuntime-7.0.1-0.i386.rpm**

2.  **rpm –ivh MQSeriesClient-7.0.1-0.i386.rpm**

3.  **rpm –ivh MQSeriesJRE-7.0.1-0.i386.rpm**

4.  **rpm –ivh MQSeriesEclipseSDK33-7.0.1-0.i386.rpm**

5.  **rpm –ivh MQSeriesSamples-7.0.1-0.i386.rpm**

6.  **rpm –ivh MQSeriesConfig-7.0.1-0.i386.rpm**

    -   Install the required code for WMQ client

    -   Compare your commands to the screen shot below.

> <img src="media/image48.png" width="511" height="599" />

1.  **Using the sample programs**

    Two sample programs that illustrate the client reconnect program are shipped with WMQ version 7.0.1. They are AMQSPHAC and AMQSGHAC, which show putting and getting respectively

<!-- -->

1.  Examine the sample program **AMQSPHAC **

> Examining the source code of the programs (fragments shown below) shows the use of the new **MQCONNX** options

<img src="media/image49.png" width="525" height="115" />

:::::::::::::

<img src="media/image50.png" width="515" height="113" />

::::::::::::::;<img src="media/image51.png" width="524" height="200" />

Also examine the callback function to be notified of reconnect events.

<img src="media/image52.png" width="524" height="404" />

1.  If you wish to review the full program you can navigate to the samples directory:

> cd /opt/mqm/samp/
>
> more amqsphac.c
>
> <img src="media/image53.png" width="401" height="290" />
>
> Make sure you do this on **mqnfs**, as we did not install the samples on **mq701**p nor **mq701s**.

1.  **Setup the command windows**

    We will need two command windows; one for the get application and one for the put application

<!-- -->

1.  Open a Gnome terminal.

> If you use an open window, make sure you are not su’ed to root. You should be logged in as “student”. If you are still root, just enter **exit**.

1.  Set the environment variable **MQSERVER with** following command

> **export MQSERVER=SYSTEM.ADMIN.SVRCONN/TCP/192.168.145.112,192.168.145.113**

-   The client uses this variable to find instances of the queue manager.

-   Observe that we now supply a list of ip addresses for the connection separated by commas. Notice that there are no spaces.

1.  In this command window start putting messages by starting the sample program AMQSPHAC.

> **/opt/mqm/samp/bin/amqsphac Q1 MQ701**

-   Notice that the sample putting program begins to put messages ( about one per second) to Q1 on qmgr MQ701.

-   Leave this command window open.

> <img src="media/image54.png" width="523" height="377" />

1.  Open another Gnome terminal and position it so that you can see both windows.

<!-- -->

1.  Set the environment variable **MQSERVER with** following command

> **export MQSERVER=SYSTEM.ADMIN.SVRCONN/TCP/192.168.145.112,192.168.145.113**

-   The client uses this variable to find instances of the queue manager.

-   Observe that we now supply a list of ip addresses for the connection separated by commas. Notice that there is no spaces.

1.  In this command window start getting messages by starting the sample program AMQSGHAC.

> **/opt/mqm/samp/bin/amqsghac Q1 MQ701**

-   Notice that the sample getting program gets all of the messages from **Q1** on qmgr **MQ701** that have been put to date and then keeps pace with the putting program, displaying each message as it arrives.

-   Leave this command window open.

> <img src="media/image55.png" width="538" height="394" />

1.  **Failing over the queue manager**

    While these two programs are running we will failover the queue manager and observe the results.

<!-- -->

1.  Verify that the two windows are still open on **mqnfs** and that the sample programs are still running, putting and getting messages.

2.  Go to the machine where the active instance of the queue manager **MQ701** is running, mq701p or mq701s. For this lab it should be **mq701p**.

<!-- -->

1.  Ensure that you logged in as userid “student”.

2.  In Gnome terminal session, stop the queue manager with the –is option to shutdown the queue manager allowing for failover. Enter the following command:

> **endmqm –i –s MQ701**

1.  After the queue manager has shut down, restart it as a standby instance using the **strmqm** command with the **–x** option.

> **strmqm –x MQ701**
>
> <img src="media/image56.png" width="532" height="385" />
>
> <img src="media/image57.png" width="516" height="143" />

**
**

1.  **Observe the results**

    Switch back to the machine where the MQ client code is running (**mqnfs** in this case). You will see something similar to the screen shots below for both command windows where the sample programs are running.

> ***AMQSPHAC***
>
> <img src="media/image58.png" width="442" height="329" />
>
> **AMQSGHAC**
>
> <img src="media/image59.png" width="442" height="328" />

-   Messages are put until the switchover. At which point the event listener code reports the connection status. After reconnection is established, the putting program goes on as before but this time the queue manager is running on the other instance.

-   NOTES: The getting program in this illustration displays all the records being put. However it is possible that some messages may be lost, because the messages are being put out of sync-point as non persistent massages.

1.  Verify where the queue manager is running by displaying MQ on both mq701p and mq701s.

<!-- -->

1.  Enter the following command on both mq701p and mq701s:

> **dspmq –m MQ701**

-   As shown below, you should see that MQ701 is no longer running on **mq701p**. But now it is running on mq701s where failed over. The standby has become active and the old active is now on standby.

> <img src="media/image60.png" width="401" height="179" />
>
> <img src="media/image61.png" width="400" height="186" />

1.  **Basic Failover Summary**

    During this failover demonstration we have:

    -   Configured an MQ client connection that identifies two instances of a queue manager as connection targets.

    -   Observed the changes required in an MQ API program to support client reconnect.

    -   Seen a failover occur and the client connected application keep running while the standby queue manager took over.

    -   The configuration is as shown below

> <img src="media/image2.png" width="624" height="524" />

MQ Explorer
===========

**Starting MQ Explorer**

> Earlier we installed the MQ client code and the MQ Explorer on the NFS server - **mqnfs**. In order to demonstrate multi-instance queue managers on MQ Explorer, we must return to **mqnfs** and start the MQ Explorer eclipse workbench.

1.  **Launch MQ Explorer**

<!-- -->

1.  On the NFS server machine **mqnfs** in a Gnome terminal logged in as “student”, enter the following command

<!-- -->

1.  **strmqcfg **

    -   This will launch the eclipse base **MQ Explorer**.

> <img src="media/image62.png" width="507" height="145" />

1.  **Exit the Welcome Screen**

    -   Initially the welcome screen will display. You may choose to examine the tutorials and Getting Started or you may go straight to administering MQ by clicking on the curved arrow icon. The welcome screen is only displayed on the first use of MQ Explorer.

> <img src="media/image63.png" width="582" height="419" />

1.  **Add the MQ701 Queue Manager**

<!-- -->

1.  In the left hand navigation bar, right-click on “Queue Managers” and select “Add Remote Queue Manager”.

> <img src="media/image64.png" width="459" height="304" />

1.  Enter the queue manager name as **MQ701**.

2.  Select Connect directly to an active instance

3.  Click the “Next” button

    <img src="media/image65.png" width="448" height="530" />

4.  Enter the connection details. Enter the IP address 192.168.145.112. Leave the port and channel values at the default.

5.  Mark the checkbox for **Is this a multi-instance queue manager?**

6.  Enter the connection details for the second instance. Enter the IP address 192.168.145.113. Leave the port and channel values at the default.

7.  Click **Finish**.

    <img src="media/image66.png" width="448" height="530" />

    -   The queue manager will be added to those managed by the **MQ Explorer**.

8.  Click on the icon to expand the **MQ701** queue manager.

9.  Click on **Queues** to see the local queue **Q1** added earlier in the lab.

> <img src="media/image67.png" width="401" height="285" />

1.  **Instance Details**

<!-- -->

1.  Right-click on the **MQ701** queue manager.

2.  Select **Connection Details**, then **Manage Instances**.

> <img src="media/image68.png" width="516" height="390" />

1.  Review **Manage Instances** window.

    -   Note that the **second instance** (192.168.145.113 – mq701s) is the one that is connected. This should be expected since the queue manager failed over from mq701p to mq701s earlier in the lab.

> <img src="media/image69.png" width="565" height="308" />

1.  Click **Test Connections**.

> <img src="media/image70.png" width="565" height="309" />

-   Note that the **first instance** (192.168.145.112 – mq701p) shows **Not available**.

-   Also note that you can reorder the list by clicking on **Move up** or **Move down**.

1.  Click **Close** to end the dialog.

<!-- -->

1.  **Add a Local Queue**

<!-- -->

1.  Right-click on **Queues**.

2.  Select **New** – **Local Queue**.

> <img src="media/image71.png" width="427" height="262" />

1.  Enter the name **Q2**. This queue will be used in the Part 7 of the lab.

2.  Press **Next**.

> <img src="media/image72.png" width="563" height="465" />

1.  Click in the **Default persistence** field and change to **Persistent**.

2.  Press **Finish**.

> <img src="media/image73.png" width="563" height="463" />

More Failover Tests
===================

> You have completed the lab, but you may want to work on this part if time permits. This section will give you more practice and further your understanding of fail-over and client reconnection.

1.  **Persistent and non-persistent messages**

    The test program AMQSPACH puts messages with the default persistence defined for the queue. Non-persistent messages DO NOT survive a failover between instances

<!-- -->

1.  Return to the mqnfs machine that connects as a client and work as **student.**

2.  Open four Gnome terminals and set the environment variable **MQSERVER** in each command window.

> Note: You may still have your open terminals running the sample programs. In this case you will only need to start open two additional terminals. If the sample programs are still running, please stop them by hitting &lt;ctrl-C&gt;. Position the windows, so you can see all four.
>
> **export MQSERVER=SYSTEM.ADMIN.SVRCONN/TCP/192.168.145.112,192.168.145.113 **

1.  In terminal 1, start the putting application with **non-persistent** messages by putting messages to **Q1**.

2.  **/opt/mqm/samp/bin/amqsphac Q1 MQ701**

3.  In terminal 2 (new terminal), start the putting application with **persistent** messages by putting messages to **Q2**.

4.  **/opt/mqm/samp/bin/amqsphac Q2 MQ701**

5.  On the machine that is running the active instance of the queue manager (should be mq701s), make sure you are logged in as student. End the queue manager with the following command allowing switchover.

> **endmqm –i –s MQ701**
>
> Note: If you forgot to restart the queue manager as **standby** on mq701p, you will get a message that says switchover is not possible.
>
> <img src="media/image74.png" width="436" height="35" />

1.  Restart the queue manager is standby mode with the following command:

> **strmqm –x MQ701**

1.  Switch back to **mqnfs**.

2.  In terminal 3, start the getting application for **non-persistent** messages from **Q1**.

3.  **/opt/mqm/samp/bin/amqsghac Q1 MQ701**

4.  In terminal 4, start the getting application for **persistent** messages from **Q2**.

5.  **/opt/mqm/samp/bin/amqsghac Q2 MQ701**

**
**

***Terminal 1 – AMQSPHAC – Q1 non-persistent Terminal 3 – AMQSPHAC – Q2 persistent*** <img src="media/image75.png" width="665" height="501" />

***Terminal 2 – AMQSGHAC – Q1 non-persistent Terminal 4 – AMQSGHAC – Q2 persistent***

1.  **Failure of the Queue Manager Host**

    The failover tests can be repeated, but instead of stopping the queue manager with the endmqm –i –s option simply kill the machine that the queue manager is running on or disable its network adapter. Failover will occur. Give it a try!

Summary
=======

In this lab we have

-   Configured 3 LINUX machines to use NFS4 file system one as host with two machines mounting the shared file system.

-   Installed WebSphere MQ Server on the two machines that have mounted the file system.

-   Configured WebSphere MQ with a queue manager that is running on both machines in an active/standby configuration.

-   Configured an MQ client connection that identifies two instances of a queue manager as connection targets.

-   Observed the changes required in an MQ API program to support client reconnect.

-   Seen a failover occur and the client connected application keep running while the standby queue manager took over.

-   Seen MQ Explorer with the new multiple instance data.

> End

**CONGRATULATIONS! **

**You have completed this hands-on lab.**

 
=

