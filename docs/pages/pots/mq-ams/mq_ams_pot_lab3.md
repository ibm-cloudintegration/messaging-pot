---
title: Protecting IBM MQ Managed File Transfer with IBM MQ AMS
toc: false
sidebar: labs_sidebar
folder: pots/MQ AMS
permalink: /mq_ams_pot_lab3.html
summary: Define protection policy for the MFT agent queues
applies_to: administrator
---

# Lab 3 - Protecting Managed File Transfer with AMS

## Overview

This lab provides an illustration of how to protect IBM MQ Managed File Transfer (IBM MQ MFT) with IBM MQ Advanced Message Security (IBM MQ AMS).  This lab will use both IBM MQ MFT and IBM MQ AMS which are included in IBM MQ Advanced.  The IBM MQ MFT and IBM MQ AMS configuration is already done on this lab's two VMWare images. You will review this configuration before performing the hands-on steps of running IBM MQ MFT transfers and defining IBM MQ AMS policies.  Beginning with IBM MQ 7.5, IBM MQ AMS setup has been simplified, and this lab reflects those simplifications.

## Introduction

IBM MQ Managed File Transfer (IBM MQ MFT) moves files reliably over IBM MQ (MQ).  IBM MQ MFT agents are long-running processes that serve as file transfer endpoints and that cooperate to transfer files. Any IBM MQ MFT agent can serve as a source and/or a destination for a file transfer.  IBM MQ MFT moves the file content in MQ messages over the data queue (SYSTEM.FTE.DATA.\<AGENT NAME\>) of the destination agent.  If a IBM MQ MFT transfer is interrupted, the unread MQ messages will remain on the data queue, and any user authorized to browse or get messages from that data queue can see the file content in those messages.

In this lab, you will use IBM MQ Advanced Message Security (IBM MQ AMS) to protect the messages at rest on IBM MQ MFT queues.  You will define IBM MQ AMS policies that employ digital certificates to sign and encrypt the file content in IBM MQ MFT messages and that restrict the users allowed to read messages from the IBM MQ MFT data queues and decrypt the MQ message data.  Users not defined in the IBM MQ AMS policy will not be able to read messages from the IBM MQ MFT data queue and decrypt the file content in those messages.

IBM MQ MFT file transfers are created by submitting commands to the command queue (SYSTEM.FTE.COMMAND.\<AGENT NAME\>) of the source agent.  Any user authorized to put messages to that queue can submit IBM MQ MFT file transfer requests.  In this lab, you will further use IBM MQ AMS to protect a IBM MQ MFT command queue.  You will create a IBM MQ AMS policy that specifies the only user allowed to put messages to the command queue and that requires and checks for a digital signature on each of those messages.

##	Installation and Configuration of IBM MQ Managed File Transfer

This lab uses two VMWare images: One runs Windows 10, and the second runs Ubuntu 14.04 64-bit.  This lab will use two MQ MFT agents: an MQ MFT agent named AGTRH runs on Linux, and the second agent named AGTMSRH runs on Windows 10.  Both of these agents share the same queue manager, named QMLIN, which runs on Linux and listens on port 1415. (The use of MQ AMS across MQ MFT agents on different queue managers is an advanced configuration not covered by this lab.) Agent AGTRH connects to queue manager QMLIN in bindings mode and agent AGTMSRH connects in client mode.
 
The Linux queue manager QMLIN serves as its own MQ MFT command queue manager.  A second queue manager named QMMFT, running on Windows 10 and listening on port 1416, serves as the MQ MFT coordination queue manager.  Both of these queue managers belong to an MQ cluster named AMSCLUSTER, for which QMMFT is a full repository. 

On Linux, the MQ MFT agents will run under the fteuser userid, and in subsequent activities, we will create MQ AMS policies only for fteuser and no other userid. 

## Installation and Configuration Review
Installation and configuration of MQ MFT and MQ AMS have been completed on the VMware images used for the lab. MQ MFT and MQ AMS are features of IBM MQ Advanced and you install them simply by selecting them as features during MQ installation.  Please take the time to review and understand the MQ MFT and MQ AMS configuration described in this section before proceeding to the subsequent hands-on section. 

### Configuration of MQ AMS
See the MQ KnowledgeCenter article entitled “Using IBM MQ AMS with IBM MQ Managed File Transfer” <https://www.ibm.com/support/knowledgecenter/SSFKSJ_9.0.0/com.ibm.mq.sec.doc/q014740_.htm> for a starting point for the following steps.  Please note that contrary to the instructions in this article, we, for our purposes, will defer creation of MQ AMS policies to a subsequent section of this lab.
 
1. Create self-signed certificates and keystores.  On both Windows 10 and Linux, start the IBM Key Management Tool with the strmqikm command, and use the graphical tool to create a self-signed certificate and keystore for each of the VMWare images.  On both platforms, the distinguished name used for this lab is “CN=fteuser, O=ibm, C=US”.  Because our Windows MQ MFT agent AGTMSRH connects to its queue manager in client mode, its keystore must be of type JKS (Java Keystore); please find it on the Windows image at C:\PoT-messaging\MQ-POT\MFT_AMS_Win7\setup\ams\keys\fteuser.jks.  Because our Linux MQ MFT agent AGTRH connects to its queue manager in bindings mode, its keystore must be of type CMS (Cryptographic Message Syntax); please find it on the Linux image at /home/fteuser/MFT_AMS_Linux/setup/ams/keys/fteuser.kdb.  Using “password” as the password, open each keystore in the IBM Key Management Tool, and examine the contents, including the certificate label and the distinguished name; then close without saving any changes.

2. Create a keystore.conf file on each platform, so that MQ AMS can find the keystore and use its certificate for signing (integrity) and encryption/decryption (privacy).  

    * On Linux, this file is located in the default location (MQ AMS automatically looks there) for the userid fteuser: /home/fteuser/.mqs/keystore.conf.
    
        Its contents are:    
        ![](./images/pots/mq-ams/amslab4-image1.png) 

    * On Windows 10, this file is located at a non-default location: **C:\PoT-messaging\MQ-POT\MFT_AMS_Win7\setup\ams\keys\keystore.conf**. We will specify this custom path as the value for the Java property MQS_KEYSTORE_CONF when we create the MQ MFT agent **AGTMSRH**.  
        The contents of this file are:    
        ![](./images/pots/mq-ams/amslab4-image2.png)
    
1. Configure the required permissions for the fteuser userid on the MQ AMS policy and error queues on the Linux queue manager QMLIN:

    *setmqaut -m QMLIN -t queue -n SYSTEM.PROTECTION.POLICY.QUEUE -p fteuser +browse*    
    
    *setmqaut -m QMLIN -t queue -n SYSTEM.PROTECTION.ERROR.QUEUE -p fteuser +put*
	
### Configuration of MQ AMS 
1. On Windows 10, run the following command to define the coordination queue manager:     
 
    *fteSetupCoordination -coordinationQMgr QMMFT -f*
    
1. On Windows 10, run the following command to define the MQ MFT queues on the coordination queue manager: 
   
    *runmqsc QMMFT < C:\ProgramData\IBM\MQ\mqft\config\QMMFT\QMMFT.mqsc*  

1. On Windows 10, run the following command to create the MQ MFT agent AGTMSRH as a Windows service, specifying the custom path of the keystore configuration file (the -sj parameter allows Java properties to be passed to the Windows service): 
    
    *fteCreateAgent -agentName AGTMSRH -agentQMgr QMLIN -agentQMgrHost rhelin  -agentQMgrPort 1414 -agentQMgrChannel SYSTEM.DEF.SVRCONN -s -su fteuser -sp password -sj -DMQS_KEYSTORE_CONF=C:/MQv75-POT/MFT_AMS_Win7/setup/ams/keys/keystore.conf*
    
1. On Linux, run the following command to define the coordination queue manager:    
 
    *fteSetupCoordination -coordinationQMgr QMMFT -coordinationQMgrHost student1 -coordinationQMgrPort 1416 -coordinationQMgrChannel SYSTEM.DEF.SVRCONN -f*

1. On Linux, run the following command to define the commands queue manager:    
 
    *fteSetupCommands -connectionQMgr QMLIN -f*
    
1. On Linux, run the following command to create the MQ MFT agent named AGTRH: 
    *fteCreateAgent -agentName AGTRH -agentQMgr QMLIN -f*
    
1. On Linux, run the following command to define the queues for the MQ MFT agent AGTRH:    
    *runmqsc QMLIN < /var/mqm/mqft/config/QMMFT/agents/AGTRH/AGTRH_create.mqsc*

1. Copy the Windows file **C:\IBM\MQv75\mqft\config\QMMFT\agents\AGTMSRH\AGTMSRH_create.mqsc** to the Linux image, then, on Linux run the following command to define the queues for agent AGTMSRH:    
    *runmqsc QMLIN < AGTMSRH_create.mqsc*

## Hands-on Lab Steps

Now that you have reviewed and understood the MQ MFT and MQ AMS configuration already set up on the two VMWare images, you are ready to proceed to the following hands-on tasks.  Firstly, you will do some start-up tasks.  Secondly, you will run MQ MFT transfers without MQ AMS involved.  Thirdly, you will create a new agent on Linux, define and run transfers between the Linux agents. You will then define MQ AMS policies for the two MQ MFT agents' data queues, then run the same file transfers with the data queues now protected.  Lastly, you will define a MQ AMS policy on the command queue of one of the MQ MFT agents, to protect the command queue from unauthorized requests and to ensure non-repudiation.

### Start-up Tasks
1. If you previously did Lab 2, your Windows 10 VM should still be running. If not already started, start the Ubuntu Linux image. Make sure both the Windows 10 and Ubuntu 14.01 VMs are running before you continue.

	![](./images/pots/mq-ams/amslab4-image0.png)
 
2. You should be logged in both VMs as ‘IBM Demo’ (ibmdemo). 
   
    ![](./images/pots/mq-ams/amslab4-image3.png) 
 
3. Enter ‘passw0rd’ for the password (on both VMs) and click ‘Log In’.    
   
    ![](./images/pots/mq-ams/amslab4-image4.png) 
 
1. Start MQ Explorer by double-clicking the icon on the desktop.
    
    ![](./images/pots/mq-ams/amslab4-image5.png) 
 
1. Open a command prompt by clicking the Terminal icon on the desktop.    
    
    ![](./images/pots/mq-ams/amslab4-image6.png) 
  
1. Set the MQ environment variables with the following command:    
   
    *. /opt/mqm/bin/setmqenv –s –l*   
    
    ![](./images/pots/mq-ams/amslab4-image7.png)
 
    This command sets the PATH and lib paths for issuing MQ commands. Make sure to leave a space between the initial period and slash mark.

1. Start queue manager QMLIN with the following command:
    
    *strmqm QMLIN*  
    
    Also start QM1.
    
    *strmqm QM1*
     
    ![](./images/pots/mq-ams/amslab4-image8.png)
 
1. Return to the Windows 10 VMware image. 

1. Continuing from the previous Lab 2, you should still be logged in as ‘ibmdemo’. If you are logged on as another user, you need to log out and log in as ibmdemo. 
 
1. Click Start.
 
1. Click the User icon, then *Sign out*. 
   
    ![](./images/pots/mq-ams/amslab4-image9.png) 
 
1. Click User icon for ‘IBM Demo’ and set Password to ‘passw0rd’.
    
    ![](./images/pots/mq-ams/amslab4-image10.png) 
 
1. Click the right arrow or hit enter.
    
    ![](./images/pots/mq-ams/amslab4-image11.png) 
 
1. Once signed on, feel free to open IBM MQ Explorer by double-clicking the icon on the desktop.
    
    ![](./images/pots/mq-ams/amslab4-image12.png) 
 
1. Your list of queue managers may look different depending on what labs you have done. 

1. QMAMS, QMMFT, and QMMSWIN should be shown, as well as remote queue manager QMLIN on ‘ubuntu(1415)’.  
  
    ![](./images/pots/fmq-ams/amslab4-image13.png) 
 
1. If those queue managers are shown, skip to Step 23.
 
1. If those queue managers are not shown, right-click Queue Managers and select Show/Hide Queue Managers. 
   
    ![](./images/pots/mq-ams/amslab4-image14.png)
 
1. Unhide the required queue managers by highlighting the queue manager(s) and select Show.
    
    ![](./images/pots/mq-ams/amslab4-image15.png) 
 
1. Select the rest of the queue managers and select Hide.
    
    ![](./images/pots/mq-ams/amslab4-image16.png) 
 
1. The queue managers to be shown should look like this:
    
    ![](./images/pots/mq-ams/amslab4-image17.png) 
 
1. Click Close. 

1. Right-click QMLIN on ‘ubuntu(1415)’ and select ‘Connect'. 
   
    ![](./images/pots/mq-ams/amslab4-image18.png)
    
    Enter "passw0rd" for the password when prompted. You should also connect QM1 in the same manner. 
 
1. Expand Queue Manager Clusters > AMSCLUSTER > Full Repositories and Partial Repositories.  
  
    ![](./images/pots/mq-ams/amslab4-image19.png)
 
1. Once the connection for QMLIN is completed, all four queue managers will now be able to communicate via the AMSCLUSTER.
 
1. Open Windows Explorer by double-clicking its icon on the desktop.    
    
    ![](./images/pots/mq-ams/amslab4-image20.png)
 
1. In Windows Explorer, verify the existence of directory **C:\\PoT-messaging\\IBM\\WMQAMS\\stewusr\\.mqs**. Verify that it has a keystore.conf file. This file defines the location of the key repository for user stewusr.    
    ![](./images/pots/mq-ams/amslab4-image21.png) 
 
1. Right-click keystore.conf and select ‘Edit with Notepad++’.    
    ![](./images/pots/mq-ams/amslab4-image22.png) 
 
1. Review the contents of the file. ‘cms.keystore’ is the path to the keystore where the user’s personal certificate is stored as well as the signer keys for users who will exchange messages. ‘cms.certificate’ is the label of the user’s personal certificate. The ‘cms’ parameters are for the server interceptor when the application is connecting via bindings. The ‘JKS’ are for the client interceptor when the application is running as an MQ client.    
    ![](./images/pots/mq-ams/amslab4-image23.png) 
 
1. Return to MQ Explorer and expand the folder Managed File Transfer. 

1. Right-click QMMFT (coordination queue manager) and select Connect.    
    ![](./images/pots/mq-ams/amslab4-image25.png) 
 
###	Start MFT agents 
1. In the command prompt window, start the MFT agents AGTMS and AGTMSRH with the following commands:
    
    *fteStartAgent AGTMS*    
    *fteStartAgent AGTMSRH* 
     
    ![](./images/pots/mq-ams/amslab4-image26.png)
 
1. In MQ Explorer > Managed File Transfer, expand QMMFT and click Agents.    
    ![](./images/pots/mq-ams/amslab4-image27.png) 
 
    Agents AGTMS and AGTMSRH should be highlighted in green (good) which means they are running and ready for any active transfers taking place.    
   
    ![](./images/pots/mq-ams/amslab4-image28.png) 
 
1. Return to the Ubuntu VMware image by clicking the tab on the taskbar.    
   
    ![](./images/pots/mq-ams/amslab4-image29.png) 
 
    Enter the password ‘passw0rd’ if the session has timed out.
     
1. In the command prompt window, switch user to fteuser by entering the following command:
    
    *su – fteuser*
 
1. Enter ‘password’ for password.     
    ![](./images/pots/mq-ams/amslab4-image30.png)
 
1. Verify the existence of directory /home/fteuser/.mqs by entering the command:    
    *ls –al*   
    ![](./images/pots/mq-ams/amslab4-image31.png)
 
1. Verify the existence of the keystore.conf file by entering the following command:    
    *ls .mqs* 
    
    View the contents of the keystore.conf file by entering the following command:    
    *more .mqs/keystore.conf*   
    ![](./images/pots/mq-ams/amslab4-image31.png)
 
    This file should look similar to the keystore.conf file for stewusr. But notice that the paths are in Linux format and this points to the keystore for fteuser. 
    
1. Start the MFT agents AGTRH and AGTRHMS by entering the following commands:                           
    *fteStartAgent AGTRH*    
    *fteStartAgent AGTRHMS* 
    
    Note: Correct case is required on Linux.    
    
    ![](./images/pots/mq-ams/amslab4-image32.png)
    
    {% include note.html content="If you receive ‘command not found’, you may have closed the terminal window or opened a new terminal window. Remember you must setup the MQ environment with the command: 
*. /opt/mqm/bin/setmqenv –s –l*. Then run the fteStartAgent commands again." %}     
 
1. You can view MQ Explorer on Linux as well as Windows. Expand its Managed File Transfer folder, select **QMMFT**, right-click, and select Connect.    
    
    ![](./images/pots/mq-ams/amslab4-image35.png) 
  
1. Upon connecting, expand QMMFT. The five subfolders shown below should be visible. Select the Agents subfolder. MQ Explorer will show that the MFT agents on Linux, AGTRH and AGTMSRH, are now running and have a status of ‘Ready’.     
    
    ![](./images/pots/mq-ams/amslab4-image36.png)
 
### MQ MFT Transfers Without MQ AMS 
1. Return to Windows. In the Managed File Transfer section of MQ Explorer, select the Transfer Templates subfolder. Right-click and select ‘New Template’.    
    
    ![](./images/pots/mq-ams/amslab4-image37.png)
 
1. On the first panel, enter “WinToLin” as the template name. Click the drop-down for source agent and select AGTMSRH. Click the drop-down for destination agent and select AGTRH. Click Next.    
    
    ![](./images/pots/mq-ams/amslab4-image38.png) 
 
1. On the next panel, click the Add button.    
    
    ![](./images/pots/mq-ams/amslab4-image39.png) 
 
1. On the resulting Add a transfer item panel, leave Mode at Binary transfer. For source, set Type to File and click Browse. Navigate to the directory ‘C:\\PoT-messaging\]\MQ-POT\\MFT_AMS_Win7\\source\\files\\’ and select **OneMillionRecords.dat**. Click Open.    
    
    ![](./images/pots/mq-ams/amslab4-image40.png) 
 
1. For destination, set Type to File and set the path to '**/tmp/OneMillionRecords.dat**'. Check the box for Overwrite files if present. Click OK.    
    
    ![](./images/pots/mq-ams/amslab4-image41.png)
 
1. On the New transfer template panel, verify that the transfer item was added. Click Finish.    
    
    ![](./images/pots/mq-ams/amslab4-image42.png)
 
1. Create a template for the converse transfer. Still in the Managed File Transfer section of MQ Explorer, select the Transfer Templates subfolder. Right-click and select ‘New Template’. On the first panel, enter '**LinToWin**' as the template name, **AGTRHMS** as the source agent, and **AGTMS** as the destination agent. Click *Next*.    
    
    ![](./images/pots/mq-ams/amslab4-image43.png) 
 
1. On the next panel, click the Add button.    
    
    ![](./images/pots/mq-ams/amslab4-image44.png) 
 
1. On the resulting Add a transfer item panel, leave Mode at Binary transfer. For source, set Type to File and type the file name '**/tmp/OneMillionRecords.dat**'. For destination, set Type to *Directory* and click Browse. Navigate to the directory to C:\\PoT-messaging\\MQ-POT\\MFT_AMS_Win7\\destination\\. Click OK.    
    
    ![](./images/pots/mq-ams/amslab4-image45.png) 
 
1. Click the checkbox to ‘Overwrite files if present’. Verify that the transfer item was added on the New transfer template. Click OK.    
    
    ![](./images/pots/mq-ams/amslab4-image46.png) 
 
1. Click the ‘Finish’ button the summary pane to save the template.    
    
    ![](./images/pots/mq-ams/amslab4-image47.png) 
 
1. In the Managed File Transfer section of MQ Explorer, under Transfer Templates, you should now have two transfer templates. Expand them to see the details. The templates should look like this:    
    
    ![](./images/pots/mq-ams/amslab4-image48.png) 
 
1. Select the WinToLin template, right-click, and select Submit. In the Managed File Transfer – Current Transfer Progress view, you should see the transfer run successfully to completion.    

	![](./images/pots/mq-ams/amslab4-image49.png) 
	
1. In the Managed File Transfer – Current Transfer Progress view, you should see the transfer run successfully to completion. The following screen capture shows the result of running the WinToLin transfer.    
    
    ![](./images/pots/mq-ams/amslab4-image50.png) 
 
1. Select the LinToWin template, right-click, and select Submit.   
    
    ![](./images/pots/mq-ams/amslab4-image51.png) 
 
1. In the Managed File Transfer – Current Transfer Progress view, you should see the transfer run successfully to completion. The following screen capture shows the result of running both the WinToLin and LinToWin transfers. 

	![](./images/pots/mq-ams/amslab4-image52.png)   
 
1. Expand the resultant transfers to view the source and destination files. Navigate to the destination directories on both images to confirm the files and the file details.    
    
    ![](./images/pots/mq-ams/amslab4-image52a.png) 
 
1. You have now verified MQ MFT file transfers without using MQ AMS between the two images. 

### MQ MFT Transfers With MQ AMS on Data Queues

1. On Linux, in the fteuser terminal, stop the MQ MFT agents AGTRH and AGTRHMS with the following command:    
 
    *fteStopAgent AGTRH*    
    *fteStopAgent AGTRHMS*    
    
    ![](./images/pots/mq-ams/amslab4-image53.png)
 
1. On Windows 10, open a command prompt and enter the following command to stop the agent AGTMSRH:    
 
    *fteStopAgent AGTMS*    
    *fteStopAgent AGTMSRH*    
    
    ![](./images/pots/mq-ams/amslab4-image54.png)
 
1. On Windows 10, in the Managed File Transfer section of MQ Explorer, select the Agents subfolder under QMMFT. MQ Explorer should show all MQ MFT agents with a status of Stopped. 
2.    
    ![](./images/pots/mq-ams/amslab4-image55.png) 
 

### MQ MFT Transfers With MQ AMS 

1. Move to the Linux system and return to the command prompt. 
	
1. Define a new security policy for agent AGTRH with the following command.    
 
    *setmqspl -m QMLIN -p SYSTEM.FTE.DATA.AGTRH -s SHA1 -a "CN=fteuser, O=ibm, C=US" -e AES128 -r "CN=fteuser, O=ibm, C=US"*    
    
    ![](./images/pots/mq-ams/amslab4-image56.png) 
 
1. In MQ Explorer, expand queue manager QMLIN, and click Security Policies. You will find the new Security Policy with Policy name **SYSTEM.FTE.DATA.AGTRH** which is the data queue for agent AGTRH. AGTRH was started by fteuser so we will use his cert. Right-click the policy and select Properties.    
    
    ![](./images/pots/mq-ams/amslab4-image57.png) 
 
1. The policy indicates that messages will be signed with SHA1, and only agents with the indicated distinguished name can put messages on the AGTRH's queue. It also will use the AES128 encryption cypher and only agents with the indicated distinguished names can receive the messages.    
    
    ![](./images/pots/mq-ams/amslab4-image58.png) 
 
1. In the command prompt start agent AGTRH.    
    
    ![](./images/pots/mq-ams/amslab4-image59.png) 
 
1. In MQ Explorer under Managed File Transfer, expand QMMFT and select Agents. You will now see that AGTRH is Ready.  
    
    ![](./images/pots/mq-ams/amslab4-image60.png) 
 
1. We will now add a policy for agent AGTMSRH which runs on Windows, so move to the Windows image.     

1. This time we will use MQ Explorer to define the policy. Expand queue manager QMLIN, right-click Security Policies and select New > Security Policy.    
    
    ![](./images/pots/mq-ams/amslab4-image61.png)     
 
1. Click Select, scroll to **SYSTEM.FTE.DATA.AGTMSRH**, highlight and click OK.    
    
    ![](./images/pots/mq-ams/amslab4-image62.png)     
 
1. Under Policy, check ‘Sign and encrypt’. Click Next.    
    
    ![](./images/pots/mq-ams/amslab4-image63.png) 
 
1. Under Signing, click the pull-down and select **SHA1** Click Add, and enter the distinguished name for the user who started agent AGTMSRH - fteuser.    
    
    ![](./images/pots/mq-ams/amslab4-image64.png)     
 
1. Click OK, then Next.
 
1. Under Encryption, click the drop-down and select AES128. 

1. Click Add, and enter the distinguished name for **fteuser** who started agent AGTMSRH:    

    *CN=fteuser, O=ibm, C=US*    
    
    ![](./images/pots/mq-ams/amslab4-image65.png)
 
1. Click OK, then Next. 

1. Review the summary page and click Finish.    
    
    ![](./images/pots/mq-ams/amslab4-image66.png) 
 
1. Pertaining to the transfer template WinToLin, we now have two security policies on QMLIN, one for the source agent AGTMSRH and one for the destination agent AGTRH.    
    
    ![](./images/pots/mq-ams/amslab4-image67.png) 
 
1. On the Windows desktop, you will find an icon for fteuser. Double-click the icon to open a command prompt with fteuser’s credentials. We want fteuser to start AGTMSRH.  
  
    ![](./images/pots/mq-ams/amslab4-image68.png)  
 
1. If prompted for a password, enter password.    
    
    ![](./images/pots/mq-ams/amslab4-image69.png) 
 
1. Start the agent AGTMSRH as before.    
 
    *fteStartAgent AGTMSRH*    
    
    ![](./images/pots/mq-ams/amslab4-image70.png) 
 
    {% include note.html content="Remember that this agent is running on the Windows system, but is connecting to queue manager QMLIN on Linux as a MQ client. Its data queue is SYSTEM.FTE.DATA.AGTMSRH on QMLIN." %}

1. MQ Explorer will now show both agents AGTRH and AGTMSRH are ready.
    
    ![](./images/pots/mq-ams/amslab4-image71.png) 
 
1. Still in MQ Explorer, select Managed File Transfer > QMMFT > Transfer Templates. Right-click WinToLin and select Submit.    
    
    ![](./images/pots/mq-ams/amslab4-image72.png) 
 
1. Click Transfer log to view the status of the transfer.    
   
    ![](./images/pots/mq-ams/amslab4-image73.png) 
 
1. After a few seconds, it will show Successful.  
    
    ![](./images/pots/mq-ams/amslab4-image74.png) 
 
1. Still on Windows, return to the ibmdemo's command prompt window and define a new security policy for AGTMS with the following command.    
 
    *setmqspl -m QMWIN -p SYSTEM.FTE.DATA.AGTMS -s SHA1 -a "CN=stewusr, O=ibm, C=US" -e AES128 -r "CN=stewusr, O=ibm, C=US"* 
    
    ![](./images/pots/mq-ams/amslab4-image74a.png)
    
1. In MQ Explorer, expand queue manager QMWIN, and click Security Policies. You will find the new Security Policy with Policy name SYSTEM.FTE.DATA.AGTMS which is the data queue for agent AGTMS. AGTMS was started by stewusr so we will be using his cert. Right-click the policy and select Properties.     
    
    ![](./images/pots/mq-ams/amslab4-image75.png)
 
1. The policy indicates that messages will be signed with SHA1, and only agents with the indicated distinguished name can put messages on the AGTMS’s queue. It also will use the AES128 encryption cypher and only agents with the indicated distinguished names can receive the messages.    
    
    ![](./images/pots/mq-ams/amslab4-image76.png) 
 
1. In the template LinToWin we indicated that agent AGTRHMS (started by stewusr on Linux) will send to AGTMS which will be started by stewusr on Windows. stewusr will be a recipient, so his distinguished name must be in the recipient list. 

1. On the Windows desktop, you will find an icon for **stewusr**. Double-click the icon to open a command prompt with stewusr’s credentials. We want stewusr to start AGTMS.  
  
    ![](./images/pots/mq-ams/amslab4-image76a.png)  
 
1. If prompted for a password, enter password. 
       
1. In **stewusr's** command prompt, start agent AGTMS as before:    

    *fteStartAgent AGTMS*    
    
    ![](./images/pots/mq-ams/amslab4-image77.png)
 
1. We still need to add a security policy for AGTRHMS’s data queue. Right click Security Policies under queue manager QMMSWIN and select New > Security Policy.    
    
    ![](./images/pots/mq-ams/amslab4-image78.png) 
 
1. On the ‘Create a security policy’ pane click Select and scroll to find the queue ‘SYSTEM.FTE.DATA.AGTRHMS’. Hightlight the queue name and click OK.    
    
    ![](./images/pots/mq-ams/amslab4-image79.png) 
 
1. Check the radio button for ‘Sign and Encrypt’ and click Next.    
    
    ![](./images/pots/mq-ams/amslab4-image80.png) 
 
1. On the next pane, click the drop-down for ‘Message signing algorithm’ and select SHA1. Check the radio button for ‘Only accept signed message from the message originators listed below’. Click Add and enter the distinguished name for **stewusr**:    
 
    *CN=stewusr, O=ibm, C=US*    
    
    ![](./images/pots/mq-ams/amslab4-image81.png) 
 
1. Click *OK*, then *Next*.

1. For encryption, click the pull-down and select AES128. Click *Add*, and enter the stewusr’s distinguished name.    
    
    ![](./images/pots/mq-ams/amslab4-image82.png) 
 
1. Click *OK*, then *Next*. 

1. Review the summary page and click *Finish*.    
    
    ![](./images/pots/mq-ams/amslab4-image83.png) 
    
    ![](./images/pots/mq-ams/amslab4-image83a.png)
 
    We now have the security policies defined for all four agents.     
 
1. On Linux, start agent AGTRHMS from stewusr’s command prompt.    
 
    *fteStartAgent AGTRHMS*    
    
    ![](./images/pots/mq-ams/amslab4-image85.png) 
 
1. On Linux or Windows MQ Explorer you will see all four agents running and ready.    

    ***Linux***    
    ![](./images/pots/mq-ams/amslab4-image86.png)
 
    ***Windows***    
    ![](./images/pots/mq-ams/amslab4-image87.png)    
 
1. On either system’s MQ Explorer, select Transfer Templates under Managed File Transfer. Right click LinToWin and select Submit.    
    
    ![](./images/pots/mq-ams/amslab4-image88.png) 
 
1. Click Transfer Log to view progress and successful completion.    
    
    ![](./images/pots/mq-ams/amslab4-image89.png) 
 
### MQ MFT Transfers With MQ AMS on Command Queue
1. On Windows, in the stewusr terminal, stop the MQ MFT agent AGTMS with the following command:    

    *fteStopAgent AGTMS*    
    
1. Verify that it has stopped by running, from the same terminal, the following command:    
 
    *fteListAgents*    
    
    ![](./images/pots/mq-ams/amslab4-image90.png) 
 
1. In ibmdemo's command prompt, define a signing (no data encryption) policy on the command queue of AGTMS by running the following command. Because a MQ MFT command message does not contain sensitive user data, we do not encrypt the message body, but our policy does require that any message put to the command queue is digitally signed by the authorized userid fteuser.    
 
    *setmqspl -m QMWIN -p SYSTEM.FTE.COMMAND.AGTMS -s SHA1 -a "CN=potuser, O=ibm, C=US" -a “CN=fteuser, O=ibm, C=US” -a "CN=stewusr, O=ibm, C=US"*     
    
    ![](./images/pots/mq-ams/amslab4-image91.png)
 
1. In MQ Explorer, you should now see a third security policy for AGTMS's command queue under QMWIN's Security Policies subfolder. Because it requires signing only, its encryption algorithm is set to None.    
    
    ![](./images/pots/mq-ams/amslab4-image92.png) 
 
1. Return to the stewusr command prompt window on Windows and start agent AGTMS.     

    *fteStartAgent AGTMS* 

1. Use the list agents command to make sure it is started.    
 
    *fteListAgents* 
    
    If AGTMS still shows ‘stopped’, wait a few seconds and issue the command again until it shows ‘ready’.     
    
    ![](./images/pots/mq-ams/amslab4-image93.png)
 
1. Still on Windows, open a command prompt for potuser by double-clicking the icon on the desktop.    
    
    ![](./images/pots/mq-ams/amslab4-image94.png) 
 
    Previously you added a security policy for potuser to sign messages being put to AGTMS’s command queue. You want to test that this policy works for potuser, we will use the RFHUtil program for the test. 

1. In potuser's command prompt, navigate to C:\Utilities\RFHutil and enter the command *rfhutil*.  
   
    ![](./images/pots/mq-ams/amslab4-image95.png)
 
1. RfhUtil starts up. On its Main tab, set the Queue Manager Name to **QMWIN**. 
1. Set the Queue Name to **SYSTEM.FTE.COMMAND.AGTMS**. 
1. Click ‘Open File’ and navigate to C:\Users\potuser. Select file Transer-XML-potuser.txt and click Open. 
    
    ![](./images/pots/mq-ams/amslab4-image96.png)
 
1. Click the Data tab at the top and click the radio button for XML to display the transfer command which will be put on the command queue. Review the command and notice that the userId stanza is filled with potuser. 
    
    ![](./images/pots/mq-ams/amslab4-image97.png)
 
    This is a transfer request command file. Notice the sourceAgent and destinationAgent stanzas. The source agent is AGTMS, so this message will be put on AGTMS’s command queue. The userid making the request is potuser and according to the security policy you just defined, potuser is an authorized signer. This transfer should complete successfully. 
    
1. Click the Main tab to return and click ‘Write Q’ to put the message on the queue. In the message area, you will see the message was sent to the command queue.      
    ![](./images/pots/mq-ams/amslab4-image98.png)
 
1. Return to MQ Explorer and click the Transfer Log.    
    ![](./images/pots/mq-ams/amslab4-image99.png) 
 
    You will see a new successful transfer submitted by potuser. 
1. You may perform this step on either Windows or Linux. Since you are now in the Windows MQ Explorer, you can continue there. Select Security Policies under queue manager QMMSWIN.  Right-click the Security Policy **SYSTEM.FTE.COMMAND.AGTMS** and select Properties.     
    ![](./images/pots/mq-ams/amslab4-image100.png)
 
1. Under Signing you will see the distinguished names of the permitted message originators. Select “CN=potuser, O=ibm, C=US” and click Remove.    
    ![](./images/pots/mq-ams/amslab4-image101.png)    
2. Click **Apply** then OK. 

1. Potuser in now no longer permitted to sign messages on AGTMS’s command queue. To test this, return to RfhUtil. 
1. Since the object **SYSTEM.FTE.COMMAND.AGTMS** has changed, RfhUtil needs to be closed and restarted. Close RfhUtil and then restart it from potuser’s command prompt. 
1. The Queue Manager Name and Queue Name fields should still be filled in. Click Open File and select the Transfer-XML-potuser.txt file. 
1. Click **Write Q** to put another transfer message on the AGTMS’s command queue. 
1. Stop the agent AGTMS by issuing the necessary command in stewusr’s command window.    

    *fteStopAgent AGTMS*    
    ![](./images/pots/mq-ams/amslab4-image102.png)
  
1. Looking at MQ Explorer > Managed File Transfer > Transfer Log, note that there are no new transfer starting or in progress. 
1. Let’s review the Windows Event Viewer to see if there were any errors when submitting the transfer. Click Start, right-click Computer and select Manage.    
    ![](./images/pots/mq-ams/amslab4-image103.png)
 
1. Expand System Tools > Event Viewer > Windows Logs and select Application.    
    ![](./images/pots/mq-ams/amslab4-image104.png) 
 
1. You will find an error for a process with user – stewusr and program – java.exe. That is the AGTMS process which was started by stewusr. The error is “Message signer is not in the list of authorized signers. This is the error we expected.    
    ![](./images/pots/mq-ams/amslab4-image105.png) 
 
    You have used MQ AMS to protect a MQ MFT command queue. 
1. Close the Event Viewer and RfhUtil. 


CONGRATULATIONS! 
You have completed this hands-on lab.



