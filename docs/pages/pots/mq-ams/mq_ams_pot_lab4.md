---
title: Protecting Message Data on the IBM MQ Appliance
toc: false
sidebar: labs_sidebar
folder: pots/mq-ams
permalink: /mq_ams_pot_lab4.html
summary: Create certificates on the appliance and use them in AMS polices
applies_to: administrator
---

# Lab 4 - Protecting Message Data on the IBM MQ Appliance

This lab introduces the student to the components of IBM MQ Advanced Message Security (MQ AMS). Students will learn about:
 
* Protecting messages on the MQ Appliance
* New Confidentiality policy

The student will exercise MQ AMS message protection in numerous scenarios including:

* Setting policies on MQ Appliance queue mangers
* Setting Confidentiality policy on a local queue 


## Key Concepts

With IBM MQ AMS V9, IBM added the Confidentiality policy to the previous “Qualities of Protection” (QOP). 

* Confidentiality: 

	IBM MQ messages are encrypted end-to-end message using a symmetric key and are not digitally signed. This reduces the amount of asymmetric key operations by half and improves performance.
	 
    Optionally the encryption keys can be reused for multiple messages going to the same recipient. The second and subsequent messages incur no asymmetric key operations which keeps performance similar to plaintext/unprotected message throughput
 
## Lab environment 
Your environment should still being running. If so skip to You should have received a URL from the instructor. Open a browser and navigate to the URL which will open a Skytap environment. The Skytap lab environment for the PoT is shown below. There are two VMware images, one Windows and one virtual MQ Appliance.    
![](./images/pots/mq-ams/amslab5-image1.png)
 
The control buttons are highlighted for each VM as well as the overall environment. You should have already started the environment by clicking the run button. You can pause each VM or the environment at any time. The stop button will shutdown the VM(s). You should not need to stop the VMs while doing this lab.

To get a console for either VM, click the monitor icon and a new browser window will open with the Windows desktop or the MQ Appliance command line. 

If you have done the previous labs, this will be familiar to you. You may already have the Windows desktop open. You can leave your applications running if you are already logged in.

Names used in this lab: 

| Host Name | IP | Purpose |
|:-----------:|:-------:|:-----------:|
|student1 | 10.0.0.4 | Windows 10 host |
|MQAppl1 | 10.0.0.3 | IBM MQ Appliance |

| Queue Manager Name| Host Address | Port |
|:-----------:|:-------:|:-----------:|
| QM1 | 10.0.0.1 | 1414 |


| Queue Name | Queue Manager | Queue Type |
|:-----------:|:-------:|:-----------:|
| AMSQ1 | QM1 | Local |


UserIDs used in this lab:   

| User-ID |	Group	| Purpose |
|:-----------:|:-------:|:-----------:|
| Administrator	| Administrators, mqm |	Windows and MQ administrator |
| msgsend	| mqusers | IBM MQ application user-ID |
| msgrecv	| mqusers	| IBM MQ application user-ID |
| msgspy	| mqusers	| IBM MQ application user-ID |

 
### Important points 
* To login to the Windows image use the Administrator user-ID 
* The password for Administrator is passw0rd 
* The password for all other users is password 
* The password for all CMS key databases and JKS keystores is password 
* User-IDs msgsend, msgrecv and msgspy each have a CMS key database and a JKS keystore already setup in their respective directories. (You created these in the previous lab.) 

### MQ environment used in this lab 
The IBM MQ environment compromises the following:

* IBM MQ v9.0.3 (all components installed) 
* IBM MQ Explorer on the desktop (icon) 
* Two queue managers:
    * QM1 (port 1414) 
* Various other MQ objects to be used for the user scenarios
 
### Programs used in this lab 
To test various scenarios with MQ AMS, the following sample applications are available:

* amqsput (IBM MQ C sample – server) 
* amqsputc (IBM MQ C sample – client) 
* amqsget (IBM MQ C sample – server) 
* amqsgetc (IBM MQ C sample – client)

These samples come with the IBM MQ product and are located in **C:\IBM\MQv75\tools\c\Samples\Bin**.
    
## Installation and configuration 
MQ AMS has been pre-installed on the VMWare images. You can view the MQ 9.0.3 Knowledge Center for installation instructions, if interested. 
For this lab, we will begin with the configuration steps.
 
## Configuration 
The MQ Appliance has already been configured with the following objects:
 
* Queue manager - **QM1**
* Queue - **AMSQ1**
* Server connection channel – **USER.SVRCONN**
* Appliance messaging group – **mqusers**
* Users in group mqusers – **msgsend, msgrecv, msgspy**

### Verify configuration on the MQ Appliance 

Configuring MQ AMS requires that you do the following:

* Create IBM MQ queues required by MQ AMS 
* Update IBM MQ configuration files (server interceptor)
 
You will now configure IBM MQ AMS on the IBM MQ Appliance MQAppl1 by following the steps outlined below:

1. You should be logged on the Windows VM as Administrator from the previous labs. 
2. Log into the MQ Appliance using admin/passw0rd.    
    ![](./images/pots/mq-ams/amslab5-image2.png) 
 
3. Synchronize the appliance date and time with the Windows VM using the “clock” command. Use the format as shown here:
     
    ***clock yyyy-mm-dd***  for setting the date        
    ***clock hh:mm:ss***    for setting time    
 
    ![](./images/pots/mq-ams/amslab5-image3.png)    
  
    If you suspend the Skytap environment or the MQ Appliance VM, the clock is also suspended. If you do suspend the appliance you should resychronize the date and time to the Windows date and time. This will help when troubleshooting and looking at logs. 

1. At the command prompt enter the command “mqcli” to get to the MQ command line interface.    
    ![](./images/pots/mq-ams/amslab5-image4.png)
 
1. Use the “dspmq” to show the queue managers on the appliance and make sure QM1 is running. If not, start it with the following command:    

    *strmqm QM1*    
    ![](./images/pots/mq-ams/amslab5-image5.png)
 
    Run “dspmq” again to verify QM1 is running.
 
1. Make sure to start the listener so we can connect to the queue manager. Start “runmqsc” using the following command:    
 
    *runmqsc QM1*     
    
    When runmqsc is ready, enter the start listener command. AMS.LISTENER is configured to listen on port 1414.    
     
    *start listener(AMS.LISTENER)*    
    ![](./images/pots/mq-ams/amslab5-image6.png)
 
    Type “end” and hit enter to exit runmqsc.
     
1. Display the messaging group mqusers with the following command:     

    *grouplist* 
		  
    Notice the users are the same suspect users we have been using in the previous labs. As you know, these users are also defined on the Windows VM and have keystores and keystore.conf files so the AMS interceptors can access them. 
    
### Configure MQ Explorer for QM1
1. Return to the Windows 10 VM.
1. If MQ Explorer is not already open, open MQ Explorer by double-clicking the icon on the desktop.    
    ![](./images/pots/mq-ams/amslab5-image7.png) 
 
1. QM1 on the MQ Appliance was already added to MQ Explorer previously, but it is hidden. To show it on MQ Explorer, right click Queue Managers and select Show/Hide Queue Managers….      
    ![](./images/pots/mq-ams/amslab5-image8.png)
 
1. On the next panel under Hidden Queue Managers, scroll down until you find QM1. Click QM1 then click Show.    
    ![](./images/pots/mq-ams/amslab5-image9.png)
 
    Click Close when done.
     
1. QM1 will now show on the Queue Managers list, but is not connected. Right click QM1 on ’10.0.0.3(1414)’, then click Connect.    
![](./images/pots/mq-ams/amslab5-image10.png)    
 
1. When you click Connect, you will be promted for the password for demouser. demouser has been configured as an appliance messaging user and authorized to connect to the MQ Appliance queue manager.
 
    Enter **passw0rd** for demouser’s password and click OK.    
    ![](./images/pots/mq-ams/amslab5-image11.png)
 
1. QM1 is now connected. It is running on the MQ Appliance MQAppl1 on IP address 10.0.0.3 and listening on port 1414. Expand QM1.    
    ![](./images/pots/mq-ams/amslab5-image12.png) 
 
1. You first need to set the permissions for the messaging group mqusers. This will allow msgsend, msgrecv, and msgspy to connect to the queue manager and put and get messages. 
1. Right click queue manager QM1, then select Object Authorities > Manage Queue Manager Authority Records….   
    ![](./images/pots/mq-ams/amslab5-image13.png)
 
1. Click the Group tab at the top of the pane, then select New.     
    ![](./images/pots/mq-ams/amslab5-image14.png)
 
1. Type “mqusers” in the Entity name field. Select Connect and Inquire under Authorities, MQI Actions.    
    ![](./images/pots/mq-ams/amslab5-image15.png) 
 
    Click OK.
     
1. Click OK again to dismiss the success message.     
    ![](./images/pots/mq-ams/amslab5-image16.png)
 
1. Verify the permissions, then close the Manage authorities records window.     
    ![](./images/pots/mq-ams/amslab5-image17.png) 
 
1. Click Queues.     
    AMSQ1 is the queue we will be protecting with MQ AMS policy Confidentiality.     
    ![](./images/pots/mq-ams/amslab5-image18.png)
 
1. Right click AMSQ1 and select Object Authorities > Manage Authority Records….    
    ![](./images/pots/mq-ams/amslab5-image19.png)
 
1. Expand Specific Profiles and select AMSQ1.    
2. Click the Group tab at the top of the window, then click New.    
    ![](./images/pots/mq-ams/amslab5-image20.png) 
 
1. Type “mqusers” in the Entity name field. Under Authorities select the MQI actions Browse, Get, Inquire, Put, then click OK.     
    ![](./images/pots/mq-ams/amslab5-image21.png)
 
1. Click OK to dismiss the success message.    
    ![](./images/pots/mq-ams/amslab5-image22.png) 
 
1. Verify the permissions, then close the Manage object authorities window.    
    ![](./images/pots/mq-ams/amslab5-image23.png) 
 
1. Repeat steps 14 - 19 for queues TEST.IN and TEST.OUT.
1. Open a Chrome by clicking the icon on the taskbar.    
    ![](./images/pots/mq-ams/amslab5-image24.png)
 
1.	Navigate to the MQ Appliance Console:    
    <https://10.0.0.3:9090/ibm-mq/console>    

1. You may get a privacy error and will need to allow for an exception. 
Click Advanced, then click Proceed to 10.0.0.3 (unsafe).     
    ![](./images/pots/mq-ams/amslab5-image25.png)
   
1. Enter admin for User name and passw0rd for password and click Login.    
    ![](./images/pots/mq-ams/amslab5-image26.png) 
 
1. Click the Tab 1 tab to open the widgets for the MQ Appliance.    
    ![](./images/pots/mq-ams/amslab5-image27.png) 
 
1. Click Add widget.     
    ![](./images/pots/mq-ams/amslab5-image28.png)
 
1. Click Chart.    
    ![](./images/pots/mq-ams/amslab5-image29.png) 
 
1. Scroll down if needed to find the new widget and click the gears to open the settings window.      
    ![](./images/pots/mq-ams/amslab5-image30.png)
 
1. Name the widget something like “Monitor Appliance”. 

    Use the pull-downs to set Resource class to Platform central processing units and Resource type to CPU performance – running queue manager. 
Resource element will be set to User CPU Time.
 
    Make sure QM1 is shown under Queue managers to monitor. Click Add queue manager.    
    ![](./images/pots/mq-ams/amslab5-image31.png) 
 
1. Click Save. (You may need to scroll down to see Save)
 
1. The widget is now ready and will dynamically chart the data when messages start flowing. 
    You may see Waiting for Data flash on the widget. The new widget then displays a graph. At first it will be a flat line as there are no messages flowing through the queue manager. As messages begin to flow during the test, you will see the relative CPU usage displayed. 
Try to position this widget so it can be observed while running the test.    
    ![](./images/pots/mq-ams/amslab5-image32.png) 
  
1. You are now ready to test the policy. 

## Test the message throughput without a protection policy. 
You will use the IH03 MQ support-pac (RFHUtil) to test the message rate on the MQ Appliance queue manager QM1, queue AMSQ1. You will run multiple tests comparing the results for: 

* No MQ AMS protection 
* MQ AMS protection policy Privacy (sign and encrypt) 
* MQ AMS protection policy Confidentiality (encrypt only) 
*MQ AMS protection policy Confidentiality (encrypt only) altering key re use count 

### Start the tool
1. Using the Windows Explorer, navigate to **C:\tools\rfhutil\MQAppliance**. 
Right-click parmtst1.txt and select **Edit with Notepad++**.     
    ![](./images/pots/mq-ams/amslab5-image33.png)
 
1. Change the highlighted values to those shown:

    | Attribute | Value | 
|:-----------:|:-------:|
| qname | AMQS1 | 
| qmgr | USER.SVRCONN/TCP/10.0.0.3(1414) |
| userID | msgrecv |
| password | password |

    Notice that the msgcount is set to 50000. That is the total number of messages that will be written to the queue.     
    ![](./images/pots/mq-ams/amslab5-image34.png)
 
    Save and close the file. 
    
1. Open new command prompt sessions for msgrecv and msgsend by double clicking their icons on the desktop.    
    ![](./images/pots/mq-ams/amslab5-image35.png) 
 
1. In each session, navigate to **C:\tools\rfhutil\MQAppliance**. 
    The driver command runs a client program that puts messages to AMSQ1 and the measure command gets the messages from the queue.
 
1. Hit Enter in the msgrecv window first to start the measure command, then hit Enter in the msgsend window to start the drvier command.    
    ![](./images/pots/mq-ams/amslab5-image36.png) 
 
### No MQ AMS protection. 
The first test puts and gets messages from AMSQ1 without any MQ AMS policy in effect. So no protection is enforced on the messages.
 
1. Open MQ Explorer and watch the Current Queue depth for AMSQ1 while you monitor the command windows. 
    You should notice that the queue depth does not go much above 100 meaning that the measuring program is keeping up with the driving program that is putting the messages.     
    ![](./images/pots/mq-ams/amslab5-image37.png)
 
1. Watch the Monitor Appliance widget to see the CPU increase.     
    ![](./images/pots/mq-ams/amslab5-image38.png)
 
1. After a few seconds all of the messages will have been retrieved from the queue and each program reports the results.    
    ![](./images/pots/mq-ams/amslab5-image39.png) 
 
    Average throughput was 1797.31 with a peak of 2573. It took 28 seconds to complete. 

1. Record these results in a notepad session. Label the results with the name of the test.      
    ![](./images/pots/mq-ams/amslab5-image40.png)
 
1. You will run the commands again, so leave the command windows open. 

### MQ AMS policy Privacy
1. Return to MQ Explorer. 
    Under QM1, right click Security Policies > New > Security Policy….    
    ![](./images/pots/mq-ams/amslab5-image41.png) 
 
1. Enter AMSQ1 for the Queue.
    Click the radio button for Sign and encrypt under Policy and Apply this policy to all messages under Toleration. 
    Click Next.    
    ![](./images/pots/mq-ams/amslab5-image42.png) 
 
1. Under Signing, click the pull-down for Message signing algorithm and select SHA256. 
    Click the radio button for Accept signed messages from any originator. In our test, msgsend is sending the messages. 
    Click Next.    
    ![](./images/pots/mq-ams/amslab5-image43.png) 
 
1. Click the pull-down for **Message encryption algorithm** and select **AES128**. 
    Click Add for Distinguished names of permitted message recipients.
    Type in msgrecv’s distinguished name from his certificate:    
        *CN=msgrecv, O=ibm, C=US*
    Case and punctuation must be exactly as shown.    
	![](./images/pots/mq-ams/amslab5-image44.png) 
 
    Click Next.

1. Review the summary, then click Finish when satisfied.    
    ![](./images/pots/mq-ams/amslab5-image45.png) 
 
1. You now have a security policy of privacy protecting messages on queue AMSQ1. 
1. Click Queues to monitor the queue depth again. Remember to click the MQ Explorer refresh occasionally while the test runs.

1. Start the measure command in msgrecv’s window. Then start the driver command in msgsend’s window. 
1. Observe the queue depth for AMSQ1 to see if it remains deeper during this run or stary about the same as last run. 
    Watch the Monitor Appliance widet to see how resources are being used.    
    ![](./images/pots/mq-ams/amslab5-image46.png) 
 
1. When the test completes observe the reports.     
    ![](./images/pots/mq-ams/amslab5-image47.png)
 
1. Record results in your notepad session. Any surprises? 

    It took 11 times longer and the average thruput was reduced by 90%! Proving that asymetric calculations are resource intensive. 
    Of course, your results will vary from these when you are running your tests.    
    ![](./images/pots/mq-ams/amslab5-image48.png) 
 
### MQ AMS protection policy Confidentiality
The next policy to test is Confidentiality. In order to change a protection policy for a queue when using MQ Explorer, you must delete the security policy then recreate it.
 
1. On MQ Explorer, click Security Policies.
1. Right click AMSQ1 and select Delete.     
    ![](./images/pots/mq-ams/amslab5-image49.png)
 
1. Click Delete to confirm you want to delete the security policy.    
    ![](./images/pots/mq-ams/amslab5-image50.png) 
 
    Click OK to dismiss the success message.     
    
1. Refresh the MQ Explorer display.     
    ![](./images/pots/mq-ams/amslab5-image51.png)
 
1. Right click Security Policies > New > Security Policy….    
    ![](./images/pots/mq-ams/amslab5-image52.png) 
 
1. Enter AMSQ1 in the Queue field as you did before. 
    This time click the Encrypt radio button under Policy. 
    Under Toleration, leave Apply this policy to all messages checked.
    Click Next.    
    ![](./images/pots/mq-ams/amslab5-image53.png) 
 
1. Under **Encryption**, click the drop-down for Message encryption algorithm and select **AES128**. 
    Note: If you used a different algorithm in previous tests, make sure to select that algorithm again so that the comparisons will be meaningful. 
    
    Click the Custom key reuse radio button then enter 1000. Key reuse is only available for the Confidentiality policy when Encrypt is selected instead of Enrypt and Sign. Since the test is putting 50,000 messages, you can start with 1,000 to see the effect of using the same key for subsequent messages when sending to the same recipient(s). 
    
    Msgrecv will be the recipient for all the messages so click Add and type in msgrecv’s distinguished name from his certificate:
    
     *CN=msgrecv, O=ibm, C=US*
        
    Case and punctuation must be exactly as shown.     
    ![](./images/pots/mq-ams/amslab5-image54.png)
 
1. Click Next.     
    ![](./images/pots/mq-ams/amslab5-image55.png)
  
1. At the Summary pane, click Finish to complete the new policy definition.    
    ![](./images/pots/mq-ams/amslab5-image56.png) 
 
1. You now have a security policy of **confidentiality** protecting messages on queue **AMSQ1**. 
1. Click Queues to monitor the queue depth again. Remember to click the MQ Explorer refresh occasionally while the test runs.
 
1. Start the measure command in msgrecv’s window. Then start the driver command in msgsend’s window. 

1.	When the test completes observe the reports.    
![](./images/pots/mq-ams/amslab5-image57.png) 
 
1. Record results in your notepad session. 
    Without signing, we doubled the thruput compared to a privacy policy and elapsed was reduced by approximately a third. Your should see similar results in your test.      
    ![](./images/pots/mq-ams/amslab5-image58.png)
 
### MQ AMS protection policy Confidentiality reuse count 10,000
You will run the test with the same settings except bumping up the **key reuse** value to **10,000**.
 
1. Repeat the steps in 3.5.4, but this time set the **Key reuse** count to **10000**. 
1. Compare results to see if increasing the key reuse count made much difference. Apparently not much difference.     
    ![](./images/pots/mq-ams/amslab5-image59.png)
 
### MQ AMS protection policy Confidentiality reuse count unlimited
You will run the test with the same settings except setting the key reuse value to unlimited. 

1. Repeat the steps in 3.5.4 except set the Key reuse count to Unlimited. 
1. Notice the queue depth stays fairly low. 
1. Compare results to see if not limiting the key reuse count made much difference. It did increase the throughput by about 40% and reduced elapsed time by about 60%.     
    ![](./images/pots/mq-ams/amslab5-image60.png)
 
 
CONGRATULATIONS! 
You have completed this hands-on lab.



