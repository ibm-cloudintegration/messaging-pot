---
title: IBM MQ Appliance Disaster Recovery (DR)
toc: false
sidebar: labs_sidebar
folder: pots/mq-appliance
permalink: /mq_appl_pot_lab8.html
summary: IBM MQ Appliance Disaster Recovery
applies_to: [developer,administrator]
---

# Lab 8 - IBM MQ Appliance Disaster Recovery (DR)

The MQ Appliance Disaster Recovery (DR) feature supports flexible DR
topologies consisting of 'live' appliances at multiple sites. DR support
is provided at the Queue Manager (QM) level, so they can be configured
independently according to DR policy -- e.g. using a simple
main/recovery site DR strategy or multi-site main/recovery strategy.

Just one network connection is required for DR between the MQ Appliances
hosting the Queue Managers. The appliances are designed to tolerate a
level of network latency between distant sites -- different than High
Availability on the MQ appliances, which tend to be in close proximity,
and are designed for automatic takeover with no data loss.

![](./images/pots/mq-appliance/lab8/image1.png)

In a simple scenario, if a 'main' site fails, the recovery process
begins to ensure business continuity. One or several of the Queue
Managers on the 'main' appliance are to be recovered on a 'recovery'
appliance, which could be many miles away. The secondary DR Queue
Managers on the remote site are converted to Primary QMs and thus become
Primary instances on the recovery appliance or appliances, as policy
dictates. Also, an HA Queue Manager (e.g.
where the HA site is 'taken out') can be DR enabled and the HA site can
be recovered to the designated recovery site.

In this lab you will configure a pair of IBM MQ Appliances to provide a
Disaster Recovery (DR) solution.

This solution provides for the situation where a complete outage or
disaster happens in a primary location and the work must be resumed
using another IBM MQ Appliance at another location.

The lab consists of:

-   Configuring the IBM MQ Appliance hardware for disaster recovery

-   Configuring a queue manager for disaster recovery

-   Performing a disaster recovery test

Note that an appliance can support a mixture of HA, DR, or standalone
Queue Managers and which can all co-exist quite happily.

Finally, it may be worth noting that 'ports' or 'interfaces' mentioned
in this lab (e.g. eth0, eth4) are mapped differently than those on the
physical MQ appliance, that may be rack-mounted in the real world. The
diagram below provides a pictorial representation:

![](./images/pots/mq-appliance/lab8/image2.png)

The M2002 introduced several additional network interfaces, including
two new 2 x 40 GB QSFP+ ports as well as two additional 10GB SPF+ ports.
  
{% include note.html content="The MQ Appliance allows either a 10GB or 40GB Ethernet port to be used for replication on each appliance to configure DR. On the physical appliance that port by default is **eth20**, on the virtual appliance it is **eth4**.
" %}

Refer to **IP Addresses** in the [Overview the IBM MQ Appliance PoT](mq_appl_pot_overview.html)
document, to determine the
network adapters and IP addresses assigned to each appliance. Pay
particular attention to **eth4** as it is the adapter used for DR on the virtual appliance.

The lab environment consists of three virtual appliances (**MQAppl1**,
**MQAppl2**, and **MQAppl3**) and a Windows environment (**Windows 10
x64**) to perform console operations, access the Web UI and run a test
application. Only MQAppl1, MQAppl2 and MQAppl3 will be used in this lab.
MQAppl3 is designated as the DR appliance. You need to create a CSIDE
environment to work based on the "*MQ Appliance PoT 9.1.4 Configured - HA
Complete - Ready for DR*" template, or you can continue to work on one of
the other templates as long as Lab 1 and Lab 2 are completed. Log on to
CSIDE and create a new environment from the template as needed.

### Start the Environment

1.  Start the environment and wait until all three virtual machines have
    started. You may shut down **MQAppl4** as it will not be used in
    this lab.

2.  The MQ appliances will have fully started when they reach the
    **login** prompt.

    ![](./images/pots/mq-appliance/lab8/image4.png)

### Configure Hardware for Disaster Recovery 

In this activity, you will verify the hardware configuration necessary
to connect the two appliances via the network.

1.  Select the first appliance (**MQAppl1**) to access the console and
    login using the administrator credentials, user-id **admin** and
    password **passw0rd**.

2.  Issue the **show ip** command to display the list of the Ethernet
    interfaces and their IP assignments (ensure you are at the **mqa\#**
    prompt).
    
    ![](./images/pots/mq-appliance/lab8/image5.png)

3.  Take note of the IP addresses for both **eth0** and **eth4**. The
    other IP addresses are not used for Disaster Recovery.

    **Note that your IP addresses may be different than the ones shown
    above.**

4.  Repeat this on **MQAppl3** and record the addresses.

    ![](./images/pots/mq-appliance/lab8/image6.png)

### Configure Disaster Recovery Queue Managers

In this activity, you will perform the configuration to make a queue
manager part of a disaster recovery configuration.

{% include note.html content="A disaster recovery configuration is setup by defining a primary and secondary queue manager. The primary appliance is **MQAppl1** and the secondary appliance is **MQAppl3**. You will use existing queue manager **QM1**. During the configuration, the queue manager must be in a stopped state. " %}
  

1.  Switch to the Windows VM and start a **Firefox** web browser using
    the icon on the Desktop.

    ![](./images/pots/mq-appliance/lab8/image7.png)

2.  Enter the IP address for **MQAppl1** (<https://10.0.0.1:9090>) to
    log onto the MQ console.

    ![](./images/pots/mq-appliance/lab8/image8.png)

    **Note:** If you get a warning about a potential security risk, click
    the **Advanced...** button, then click **Accept the Risk and
    Continue**.

3.  Sign in with **admin** / **passw0rd**.
    
    ![](./images/pots/mq-appliance/lab8/image9.png)

	We will use QM1 which was created previously.

    QM1 uses port 1414 and has a file system size of 2GB. Remember that
    the available file system on the virtual appliance is very limited.

4.  To prepare the disaster recovery configuration, the queue manager
    must be stopped. If QM1 is running, highlight it in the **Local
    Queue Managers** widget and stop it by clicking the **Stop** icon.
    
    ![](./images/pots/mq-appliance/lab8/image10.png)
    
5. Click **Stop** to confirm QM1 to end.

    ![](./images/pots/mq-appliance/lab8/image11.png)

6. Refresh the queue manager status to make sure it is stopped.

    ![](./images/pots/mq-appliance/lab8/image12.png)

7. With **QM1** highlighted, click
    ![](./images/pots/mq-appliance/lab8/image13.png) (*More actions* menu) and then **Disaster Recovery...**.

    ![](./images/pots/mq-appliance/lab8/image14.png)

8. Click the **Create DR primary** option.
    
    ![](./images/pots/mq-appliance/lab8/image15.png)
    
9. MQAppl3 will be the recovery appliance, so fill in the fields as
    shown below:
    
    **Standby appliance name: MQAppl3**
    
    **Standby appliance IP address: 10.0.4.3** (or appropriate *eth4*
    address of *MQAppl3*)

    **DR port: 2014**

    ![](./images/pots/mq-appliance/lab8/image16.png)

	Note that 10.0.4.3 is the IP address of eth4 (physical eth20)
    network adapter on MQAppl3 (refer to table at the top of this
    guide).
    
10. Click **Create**.

	 {% include note.html content="The information used to generate the command: 
    
    
    **crtdrprimary -m QM1 -r MQAppl3 -i \<MQAppl3-eth4-IP\_Address\> -p 2014** 
    
    
    * The queue manager name **(-m)** is passed to the command along with the name (**-r**) and IP address (**-i**) of the secondary (recovery) appliance. A port number (**-p**) used for replication must be specified.
     
    * The port number must be in the 1025-9999 range, unused, but not 2222.
    " %}
    
	If the command is successful, you will receive a green message
    indicating that the queue manager is prepared for disaster recovery
    replication.

	On successful completion of the command, the command generates and outputs the associated **crtdrsecondary**
    command to be run **AS IS** on the secondary (**MQAppl3**) appliance.
    Take note of that command as you will re-use it in the next step.
    
    ![](./images/pots/mq-appliance/lab8/image17.png)
    
11. **Copy** the command in the grey "**Create DR secondary command**"
    box, as you will need to run it on the recovery appliance *MQAppl3*.
    
    ![](./images/pots/mq-appliance/lab8/image17a.png)

12. Click **Finish**. 

	If you receive the error below, contact your instructor for help in
    making space available.
    
    ![](./images/pots/mq-appliance/lab8/image18.png)
    
13. Open a new browser tab and enter the IP address for **MQAppl3**
    (<https://10.0.0.3:9090>) to log onto the MQ console.

    Note: If you get a warning about a potential security risk, click
    the **Advanced...** button, then click **Accept the Risk and
    Continue**.
    
    ![](./images/pots/mq-appliance/lab8/image19.png)

14. Log in with **admin** / **passw0rd**.

15. Click the **Disaster Recovery** button on the top right of the page, and then click **Create DR secondary**.
    
    ![](./images/pots/mq-appliance/lab8/image20.png)

16. Paste the *copied* **crtdrsecondary** command in the "**Paste command here**"
    box on appliance **MQAppl3**.

    Once you paste the command, the other fields will automatically fill
    in.

	For example: **crtdrsecondary -m QM1 -s 2048 -l MQAppl1 -i 10.0.4.1
    -p 2014**
    
     ![](./images/pots/mq-appliance/lab8/image21.png)
    
17. Click **Create**.

	You should see a green confirmation box at the top of the console. 
	
	 ![](./images/pots/mq-appliance/lab8/image21a.png)

	You can click the **X** to dismiss the box.
	    
18. You should see **QM1** show up in the **Local Queue Managers**
    widget. You may need to click the refresh button for **Local Queue
    Managers** if you get an error message.
    
    ![](./images/pots/mq-appliance/lab8/image22.png)

19. Check the DR status. Highlight **QM1**, then click ![](./images/pots/mq-appliance/lab8/image13.png) and **Disaster Recovery...**. Click **DR status** from the popup menu.
    
    ![](./images/pots/mq-appliance/lab8/image23a.png)

20. QM1 should have a **Normal** status. Click **Cancel**.

    ![](./images/pots/mq-appliance/lab8/image24.png)
     
     {% include note.html content="The information used to generate the command: 
    
    
	The **crtdrsecondary** command creates a secondary version of the queue manager on the recovery appliance in a disaster recovery configuration. All parameters are supplied by the equivalent **crtdrprimary** command and should be entered exactly as shown in the output from that command. After this command is run, synchronization of data from the main to the recovery appliance begins. The queue manager status is shown as stopped, and initial synchronization progress can be followed by using the **status** command. "
	%}
  
21. Run the **status** command to inquire about the synchronization
    status.

	This command can be run on either appliance.

    Remember you must be at the **mqcli** shell.

    Command entered on **MQAppl3**:
    
    **`status QM1`**

    ![](./images/pots/mq-appliance/lab8/image25.png)

    Command entered on **MQAppl1**:

    **`status QM1`**

    ![](./images/pots/mq-appliance/lab8/image26.png)

22. While at the **MQAppl1** appliance command line, start queue manager
    **QM1** on appliance **MQAppl1** in **mqcli** mode:

	**`strmqm QM1`**

    ![](./images/pots/mq-appliance/lab8/image27.png)

The DR configuration for queue manager QM1 is now complete. From
    this point on, both MQ definitions and data are asynchronously
    replicated to the secondary appliance.

### Perform a Disaster Recovery Test 

In this activity, you will perform a Disaster Recovery test to validate
the disaster recovery configuration that you just put in place.

The test includes the following steps:

-   Create a messaging user-id

-   Generate some MQ object definitions for the messaging application

-   Generate some persistent message traffic to queue manager **QM1**

-   Shutdown of the primary **MQAppl1** appliance

-   Recovery of the **QM1** queue manager on the secondary **MQAppl3**
    appliance

-   Get messages previously sent from queue manager **QM1**

Now let's start the test.

1. Return to the **MQAppl1** appliance.

2. Create a messaging user on the appliance

    **`usercreate -u potuser -g users`**

   ![](./images/pots/mq-appliance/lab8/image28.png)
   
   If you receive an **AMQ6618: The userid 'potuser' already exists message**, you can ignore it and continue.
   
   {% include note.html content="The commands are stored in a text file in C:\\Lab08\\Commands\\Commands Snippets.txt. 
    
    
   You may copy and paste the commands if you don't want to type them. However, you cannot paste in the mqcli command line, so you will need to log on to the appliance using putty, which will allow you to paste the commands. 
	" %}
    
  
3. Create some MQ objects on the appliance with the following commands:

	```
	runmqsc QM1
	
	def channel(APP.CLIENT) chltype(SVRCONN)
	
	def ql(TEST.Q)
    ```
    
    ![](./images/pots/mq-appliance/lab8/image30.png)
    
4. Grant authorizations to the MQ objects on the appliance.

	```
	set authrec objtype(qmgr) principal('potuser')
    authadd(connect,inq)
    
    set authrec objtype(queue) profile(TEST.Q)
    principal('potuser') authadd(put,get)
    
    end
    ```
    
    ![](./images/pots/mq-appliance/lab8/image31.png)
    
5. Switch to the Windows environment (**Windows 10 x64)** and double
    click the **potuser** icon. This will open a Windows command prompt.

    ![](./images/pots/mq-appliance/lab8/image32.png)
    
6. Send some persistent messages to queue manager **QM1**.
	
	From the Windows command prompt:
	
	```
	cd \Utilities\MA01
	
	set MQSERVER=APP.CLIENT/TCP/<MQAppl1-eth0-IP>(1414)
	```

	Replace \<MQAppl1-eth0-IP\> with the correct IP address -- normally
    10.0.0.1.

7. Now we use the **q** program to put some persistent messages.
	
	`q -o TEST.Q -ap -mQM1 -lmqic`

	*parameters:* 
	
	*-o output queue name (small o)*

	*-a options: p=persistent*

	*-l library: mqic=32bit (small l)*

	![](./images/pots/mq-appliance/lab8/image33.png)

8. Enter **three** messages.

9. Finally, **CTRL-C** out of the program.

10. Shut down the **MQAppl1** appliance using the Skytap dashboard
    shutdown button. Wait a minute or so for the action to work.

    ![](./images/pots/mq-appliance/lab8/image34.png)

11. Switch to the **MQAppl3** appliance and display the status of queue
    manager **QM1**.

    **`status QM1`**

    ![](./images/pots/mq-appliance/lab8/image35.png)
    
    You should see a DR status now of **Remote appliance(s)
    unavailable** as the unavailability of
    MQAppl1 is recognized.
    
    If you do not see this status, wait a few seconds and try again.

12. In the browser session for **MQAppl3**, notice that **QM1** is still
    **Stopped**. The DR functionality does not automatically start the
    queue manager.

    Highlight **QM1**, click ![](./images/pots/mq-appliance/lab8/image13.png) and then **Disaster Recovery...**. Select **DR Status**.

    ![](./images/pots/mq-appliance/lab8/image36.png)

    ![](./images/pots/mq-appliance/lab8/image23a.png)

13. Notice the status. Click **Cancel**.
    
    ![](./images/pots/mq-appliance/lab8/image38.png)

14. Make queue manager **QM1** on MQAppl3 primary.

	From the **MQAppl3** command line, enter the command:

    **`makedrprimary -m QM1`**
    
    ![](./images/pots/mq-appliance/lab8/image39.png)

    **OR**

    From the MQ Console, with **QM1** highlighted, click ![](./images/pots/mq-appliance/lab8/image13.png) and then **Disaster Recovery...** again. 

    ![](./images/pots/mq-appliance/lab8/image40.png)

    Click **Make DR primary**

    ![](./images/pots/mq-appliance/lab8/image37.png)

    You should receive a green bar with the **Successfully performed
    disaster recovery operation** message. You can click on the **X** to
    dismiss the message.

    ![](./images/pots/mq-appliance/lab8/image41.png)
    
15. Start queue manager **QM1** on MQAppl3.

	From the command line:

    **`strmqm QM1`**

    ![](./images/pots/mq-appliance/lab8/image42.png)
    
    **OR**

	From the MQ Console:
   
    With **QM1** highlighted, click the
    ![](./images/pots/mq-appliance/lab8/image43.png) button.

	![](./images/pots/mq-appliance/lab8/image44.png)
	
	
	 The queue manager has now been recovered on the secondary appliance and it is ready to accept connections and process messages.
 

16. Switch back to the Windows environment (**Windows 10 x64**) browser
    session for **MQAppl3** if necessary.

17. Click **Add Widget** and add a widget for **Queues** on **QM1**.

    ![](./images/pots/mq-appliance/lab8/image45.png)

    Re-position the queues widget next to the *Local Queue Managers*
    widget.

    Note that **QM1** still has *three* messages on the **TEST.Q**.

    ![](./images/pots/mq-appliance/lab8/image46.png)

    There may be messages on other queues left from previous labs too.

18. Browse the messages by highlighting **TEST.Q** and clicking the
    ![](./images/pots/mq-appliance/lab8/image47.png) icon on the tool bar.

    ![](./images/pots/mq-appliance/lab8/image48.png)
    
    These messages will look familiar, as they are the ones you created
    on **QM1** on **MQAppl1**.

    ![](./images/pots/mq-appliance/lab8/image49.png)

19. Click **Close**.

20. Retrieve messages previously put to the **TEST.Q** queue.

	Return to the Command Prompt and repoint the MQSERVER variable to
    the recovery appliance (at your **eth0** IP address).

    **`set MQSERVER=APP.CLIENT/TCP/<MQAppl3-eth0-IP>(1414)`**

    Replace \<MQAppl3-eth0-IP\> with the correct IP address -- normally
    10.0.0.3.

    Retrieve the messages:

    **`q -ITEST.Q -mQM1 -lmqic`**

	*parameters:*
	
	*-I input queue name (**capital i**)*

	*-a options: p=persistent*

	*-l library: mqic=32bit (small l)*

	![](./images/pots/mq-appliance/lab8/image50.png)

	**OOPS!** What happened? 2035... Security error, not authorized!
    Let's investigate....

21. On **MQAppl3**'s MQ Console, click the **Administration** icon.
    
    ![](./images/pots/mq-appliance/lab8/image51.png)

22. Expand the **Main** tab and select **File Management**.

    ![](./images/pots/mq-appliance/lab8/image52.png)
    
23. Navigate to **mqerr: \> qmgrs \> QM1**, then click on
    **AMQERR01.LOG**. The log file opens in a new tab in your browser.
    
    ![](./images/pots/mq-appliance/lab8/image53.png)

24. Scroll to the bottom and review the error.

    ![](./images/pots/mq-appliance/lab8/image54.png)

	The problem is that messaging users are **not** replicated
and must be re-created during DR time. Note that MQ authorizations are
part of the queue manager and are replicated.

25. **Close** the error log by closing the browser tab.

26. Re-create the messaging user-id **potuser** on MQAppl3.

	Switch back to the **MQAppl3** command line and enter:

	**`usercreate -u potuser -g users`**

	![](./images/pots/mq-appliance/lab8/image55.png)

27. Switch back to the Windows environment.

28. Again attempt to retrieve the messages in the potuser command
    session:

    **`q -ITEST.Q -mQM1 --lmqic`**

    *parameters:* 
    
    *-I input queue name (**capital i**)*

    *-a options: p=persistent*

    *-l library: mqic=32bit (small l)*

    ![](./images/pots/mq-appliance/lab8/image56.png)

	Voila! The DR test is successful and complete!

### Remove a queue manager from a disaster recovery configuration

1. Switch to the **MQAppl3** browser tab and stop queue manager
    **QM1**, by clicking ![](./images/pots/mq-appliance/lab8/image57.png) **stop**.
    
    ![](./images/pots/mq-appliance/lab8/image58.png)

2. Confirm by clicking **Stop**.

    ![](./images/pots/mq-appliance/lab8/image59.png)

3. Once QM1 is stopped, highlight **QM1**, then click
    ![](./images/pots/mq-appliance/lab8/image60.png) and then **Disaster Recovery...**. Then click **Make DR
    Secondary**.

    ![](./images/pots/mq-appliance/lab8/image61.png)

    You will receive the green operation successful message.
    
    ![](./images/pots/mq-appliance/lab8/image62.png)

    Click **X** to dismiss the message.

4. From the Skytap dashboard, start **MQAppl1**.

    ![](./images/pots/mq-appliance/lab8/image63.png)

5. Wait for MQAppl1 to initialize. You will know initialization is
    complete when the login: prompt is displayed on the MQAppl1 console
    or you are prompted to refresh the browser on the MQAppl1 browser
    tab.

    ![](./images/pots/mq-appliance/lab8/image64.png)

    ![](./images/pots/mq-appliance/lab8/image65.png)

6. Login and start **QM1** on **MQAppl1**.
    
    ![](./images/pots/mq-appliance/lab8/image66.png)

7. **QM1** will display a **Starting** status, then **Running**.
    
   ![](./images/pots/mq-appliance/lab8/image67.png)

8. Click **Add Widget** for **Queues** on queue manager **QM1**, if
    this widget is not already present.

   ![](./images/pots/mq-appliance/lab8/image68.png)

9. Reposition the **Queues on QM1** next to the widget for **Local
    Queue Managers**. You will see that the three persistent messages
    are still on **TEST.Q**.
    
    ![](./images/pots/mq-appliance/lab8/image69.png)

10. Highlight **TEST.Q** then click
    ![](./images/pots/mq-appliance/lab8/image70.png) to browse the messages.

    ![](./images/pots/mq-appliance/lab8/image48.png)

11. You have verified that your messages are still on **TEST.Q** on
    queue manager **QM1** when QM1 has been restored on the main
    production appliance. Click **Close**.
    
    ![](./images/pots/mq-appliance/lab8/image49.png)

12. On **MQAppl3**'s MQ Console, highlight **QM1** then click
    ![](./images/pots/mq-appliance/lab8/image71.png) and then **Disaster Recovery...** and then **Delete
    DR secondary**.

    ![](./images/pots/mq-appliance/lab8/image72.png)
    
    You will receive the green successful operation message.

    ![](./images/pots/mq-appliance/lab8/image73.png)

    Click the **X** to dismiss the message.

13. Wait a few seconds and you will see *QM1* disappear from the MQ
    Console. QM1 has been removed from the disaster recovery configuration.
    
    ![](./images/pots/mq-appliance/lab8/image73a.png)

14. On **MQAppl1**'s MQ Console, stop queue manager **QM1**.
    
    ![](./images/pots/mq-appliance/lab8/image74.png)

15. Confirm by clicking **Stop**.

    ![](./images/pots/mq-appliance/lab8/image75.png)

16. Once *QM1* is stopped, highlight **QM1**, then click
    ![](./images/pots/mq-appliance/lab8/image76.png) and **Disaster Recovery...** and then **Delete
    DR primary**.
    
    ![](./images/pots/mq-appliance/lab8/image77.png)

17. You will receive the green successful operation message. QM1 is now
    completely removed from DR configuration on any appliance. Click the
    **X** to dismiss the message.

    ![](./images/pots/mq-appliance/lab8/image78.png)

## Configuring disaster recovery for a high availability queue manger

You can specify that a high availability queue manager also belongs to a
disaster recovery configuration.

The following exercise provides insight into DR support for High
Availability Queue Managers and allows us to explore recovery of Queue
Managers running on designated HA appliances (e.g. at one site), to a
remote Disaster Recovery site (until such point as the main site is
recovered).

Both appliances in a high availability pair are typically located in the
same data center. If some disaster befalls the data center, and both
appliances are unavailable, you can manually start the queue manager on
a recovery appliance located in another data center. In this
environment, assume MQAppl3 is in another data center and will be the
disaster recovery appliance.

![](./images/pots/mq-appliance/lab8/image79.png)

The MQ Appliance DR feature supports flexible DR topologies consisting
of 'live' appliances at multiple sites. DR support is provided at the
Queue Manager (QM) level, so they can be configured independently
according to DR policy -- e.g. using a simple main/recovery site DR
strategy or multi-site main/recovery strategy.

Just one network connection is required for DR between the MQ Appliances
hosting the Queue Managers. The appliances are designed to tolerate a
level of network latency between distant sites -- different than High
Availability MQ appliances, which tend to be in close proximity, and are
designed for automatic takeover with no data loss.

In a simple scenario, if a 'main' site fails, the recovery process
begins to ensure business continuity. One or several of the Queue
Managers on the 'main' appliance are to be recovered on a 'recovery'
appliance, which could be many miles away. The secondary DR Queue
Managers on the remote site are converted to Primary QMs and thus become
Primary instances on the recovery appliance or appliances, as policy
dictates. In the latest update to DR support, a HA Queue Manager (e.g.
where the HA site is 'taken out') can be DR enabled and the HA site can
be recovered to the designated recovery site.

Note that an appliance can support a mixture of HA, DR or standalone
Queue Managers, which can co-exist quite happily.

Finally, it may be worth noting that 'ports' or 'interfaces' mentioned
in this lab (e.g. eth0, eth4) are mapped different to those on the
physical MQ appliance, that may be rack-mounted in the real world.

The diagram below provides a pictorial representation:

![](./images/pots/mq-appliance/lab8/image80.png)

Note that on the physical MQ appliance, the physical port (interface)
numbers are different to those ports shown in this hands-on lab, using
the Virtual Appliance. The mappings of the virtual ports are in
parentheses.

### Disaster Recovery sequence of events

The following diagrams provide the detailed sequence of events in
graphical form between the main appliance and fail over appliance in a
disaster recovery configuration.

![](./images/pots/mq-appliance/lab8/image81.png)

![](./images/pots/mq-appliance/lab8/image82.png)

![](./images/pots/mq-appliance/lab8/image83.png)

![](./images/pots/mq-appliance/lab8/image84.png)

![](./images/pots/mq-appliance/lab8/image85.png)

![](./images/pots/mq-appliance/lab8/image86.png)

![](./images/pots/mq-appliance/lab8/image87.png)

![](./images/pots/mq-appliance/lab8/image88.png)

![](./images/pots/mq-appliance/lab8/image89.png)

![](./images/pots/mq-appliance/lab8/image90.png)

### Prepare for HA group in DR configuration

If **QM1** was deleted in the previous exercise, you can skip the first two-three steps (depending on if QM1 is deleted from both appliances).

1. Since space is limited on virtual appliances, you need to delete
    **QM1** as we will not use it in this exercise and need the space.

    On **MQAppl1**, use the MQ Console and highlight **QM1** in the *Local Queue Managers* widget, and click the trash can icon.

    ![](./images/pots/mq-appliance/lab8/image91.png)

2. Click **Delete** to confirm the deletion of QM1.

    ![](./images/pots/mq-appliance/lab8/image92.png)

3. Logon on to the **MQAppl2** MQ Console and repeat the previous step to delete the
    queue manager **QM2** (if it exists) on **MQAppl2**. Make sure the
    only queue managers you see on MQAppl2 are HAQM1 and HAQM2. Delete
    any others.

    ![](./images/pots/mq-appliance/lab8/image93.png)

4. Staying on **MQAppl2**'s MQ Console, add a local queue named
    "**TEST2**" on the **Queues on HAQM2** widget (add the widget first
    if necessary).

    ![](./images/pots/mq-appliance/lab8/image94.png)

    ![](./images/pots/mq-appliance/lab8/image95.png)

5. Open the properties for **TEST2** by clicking the **Properties**
    icon.

    ![](./images/pots/mq-appliance/lab8/image96.png)

6. Change the **TEST2** queue default persistence to be  **persistent**. 

    Click **Save** to save the change, then clock **Close**.

    ![](./images/pots/mq-appliance/lab8/image97.png)    

7. Click the **Put message** icon and put **three** messages on
    **TEST2**.

    ![](./images/pots/mq-appliance/lab8/image98.png)

8. Click the **Folder** icon to browse the messages. Click **Close** after browsing them.

   ![](./images/pots/mq-appliance/lab8/image99.png)

   You are now ready to test DR.

### Configure DR for HA queue manager

1. On the **MQAppl1** MQ Console, click **HAQM1** to select it, then click
    **Stop** to end the queue manager **HAQM1**.

    ![](./images/pots/mq-appliance/lab8/image100.png)
    
    Click **Stop** to confirm you want to stop HAQM1.

    ![](./images/pots/mq-appliance/lab8/image101.png)

2. Click **Refresh** to make sure it is stopped and shows *red*.

    ![](./images/pots/mq-appliance/lab8/image102.png)

3. Repeat the previous steps to stop **HAQM2** on **MQAppl2**.

    ![](./images/pots/mq-appliance/lab8/image103.png)

### Create a DR primary and backup QM on recovery device

The next task is to create a DR primary (MQAppl2) queue manager and a
backup queue manager on the recovery site (MQAppl3). Note that the HA
appliance pair MQAppl1 and MQAppl2 need to be running also.

1. Make sure that the HA queue managers are not in split-brain status.

    On **MQAppl2**'s command line, enter the command:

    **`status HAQM2`**

    ![](./images/pots/mq-appliance/lab8/image104.png)

    If the HA status shows **Normal**, skip to Step 6.

    In this instance, the HA status for **HAQM2** shows **Partitioned**
    which is also called split-brain. The amount of data which is out of
    sync is shown to be 137284KB (what you see might differ slightly).
    This is easily resolved.

2. On **MQAppl2**'s MQ Console, highlight **HAQM2**, click
    ![](./images/pots/mq-appliance/lab8/image71.png) then **High Availability...**, and then **Resolve
    partitioned state**.

   ![](./images/pots/mq-appliance/lab8/image105.png)

   ![](./images/pots/mq-appliance/lab8/image106.png)
   
3. You will receive the green successful operation message. Click the
    **X** to dismiss the message.
    
    ![](./images/pots/mq-appliance/lab8/image106a.png)

4. Check the status for **HAQM2** again on MQAppl2's command line.
    Status is now **Normal**.

    ![](./images/pots/mq-appliance/lab8/image107.png)

5. Repeat Steps 2 - 4 on **MQAppl1** for **HAQM1** to resolve the
    partitioned status.

6. On the **MQAppl2** browser session, select **HAQM2** (stopped) and
    click the
    ![](./images/pots/mq-appliance/lab8/image71.png) pulldown and click **Disaster
    Recovery...**

    ![](./images/pots/mq-appliance/lab8/image108.png)

7. Click **Create DR Primary**.

   ![](./images/pots/mq-appliance/lab8/image109.png)

8. Specify that the **HAQM2** queue manager is the primary instance in a
    disaster recovery configuration and include a floating IP address
    that can be used by either of the appliances in the HA pair.

    Complete the dialog (below) with the IP address of eth4 from the DR
    site (MQAppl3) in the information as follows:

    **Standby appliance name: MQAppl3**
    
    **Standby appliance IP address: 10.0.4.3** (or appropriate eth4
    address of MQAppl3)

    **DR port: 2023**

    **Floating IP address: 10.0.4.4**

    ![](./images/pots/mq-appliance/lab8/image110.png)

    Click **Create**. 
    
    This generates and runs the *crtdrprimary* command.
    The crtdrprimary command configures the queue manager on both
    appliances in the DR pair, and reserves storage for the data
    snapshot on both appliances. The command would look like this if
    entered manually:

    **`crtdrprimary -m HAQM2 -r MQAppl3 -i 10.0.4.3 -p 2023 -f 10.0.4.4`**

	The parameters are as follows

	-   *-m*  the HA queue manager

	-   *-r*  name of the recovery appliance

	-   *-i*  ip address of the recovery appliance

	-   *-p*  port number of the queue manager on the recovery appliance

	-   *-f*  floating IP address

9. The floating IP address is an IPv4 address that is used to replicate
    queue manager data from *whichever HA appliance the queue manager is
    currently running on* to the queue manager on the recovery appliance.
    The floating IP address must be in the same subnet group as the
    static IP address assigned to the replication port -- eth4 (eth20
    for physical appliances) -- on both appliances. It also must be at
    least one greater than eth4's address and unused.

    Note that you do not physically configure an Ethernet port with this
    address. Select a free IP address in the same subnet as the
    replication ports on the two appliances, and specify it in the
    crtdrprimary command to make it the IP used for replication with the
    recovery appliance. You must specify a different floating IP address
    for each of the HA queue managers that you configure disaster
    recovery for.

    The DR creation task can take a bit of time, wait at least a minute
    or two.

    Do **not** click **Finish**. The crtdrprimary command returns a
    crtdrsecondary command when it has completed, for example:

    **crtdrsecondary -m HAQM2 -s 2048 -l MQAppl2 -i 10.0.4.4 -p 2023**

   ![](./images/pots/mq-appliance/lab8/image111.png)

10. **Copy** the **crtdrsecondary** command.

11. Switch to the **MQAppl3** browser tab or open a new tab for MQAppl3
    and login.

    In the top right corner of the MQ Console, click **Disaster Recovery** and then click **Create DR secondary**.
    
    ![](./images/pots/mq-appliance/lab8/image112.png)

12. **Paste** the command (you copied from MQAppl2) into the "**Paste
    command here**" box. The rest of the fields will be automatically
    filled.

    Click **Create**.

    ![](./images/pots/mq-appliance/lab8/image113.png)

13. Wait for the command to complete (a few seconds) and you will see
    that **HAQM2** is added to the *Local Queue Managers* widget.
    
    ![](./images/pots/mq-appliance/lab8/image114.png)
    
    You will get the green box with the success message that you can close.

14. Return to the **MQAppl2** browser and click the **Finish** button.

    Then highlight **HAQM2**, then click
    ![](./images/pots/mq-appliance/lab8/image115.png) and **Disaster Recovery...** and then **DR
    status**.

    ![](./images/pots/mq-appliance/lab8/image116.png)
    
15. The DR status for HAQM2 will show **Normal** (if it shows
    'Synchronization in progress', wait a few seconds for it to
    complete).

16. In the popup window, click **HAQM2** to display the DR details. The display will reflect
    what you entered on the DR Create Primary command. Click **Cancel**
    to dismiss the status panel.

    ![](./images/pots/mq-appliance/lab8/image117.png)

    ![](./images/pots/mq-appliance/lab8/image118.png)
    
17. Repeat the DR Status command on **MQAppl3**'s MQ Console.

    Highlight **HAQM2**, then click
    ![](./images/pots/mq-appliance/lab8/image119.png) **Disaster Recovery...** followed by **DR
    status**.

    Click **HAQM2** to show the details.

    ![](./images/pots/mq-appliance/lab8/image120.png)

	Notice that MQAppl3's role is DR *secondary* and the DR address is the
floating IP address you entered on the command.

	![](./images/pots/mq-appliance/lab8/image121.png)

18. Click **Cancel** to dismiss the status display.

### Test DR for HA group

You can simulate a disaster at the primary data center by shutting down
the MQAppl1 and MQAppl2 virtual appliances.

1. On the **Skytap** dashboard, shut down **MQAppl1** first, then
    shutdown **MQAppl2**. You only need to wait between shutdowns as
    long as it takes Skytap to return control.

    ![](./images/pots/mq-appliance/lab8/image122.png)

2. Return to the **MQAppl2** MQ Console. Wait for the "*Lost
    communication with the server*" message.

    ![](./images/pots/mq-appliance/lab8/image123.png)
    
3. Now you can return to the **MQAppl3** MQ Console.

    Highlight **HAQM2**, then click
    ![](./images/pots/mq-appliance/lab8/image71.png) and **Disaster Recovery...** and then **Make DR
    primary**.

    ![](./images/pots/mq-appliance/lab8/image124.png)

4. You will receive the green successful operation message. HAQM2 is
    now primary on MQAppl3 and can be started. Before you start the
    queue manager, check the DR status once more to see that it in fact
    shows DR primary.

    The status display shows that the remote appliance is no longer
    available and the recovery appliance is now the primary. Dismiss the
    display by clicking **Cancel.**
    
    ![](./images/pots/mq-appliance/lab8/image124a.png)

5. Now start the queue manager **HAQM2** on the recovery appliance
    **MQAppl3**.

    ![](./images/pots/mq-appliance/lab8/image125.png)

    Once it is started, display its DR status:

    ![](./images/pots/mq-appliance/lab8/image126.png)
    
    ![](./images/pots/mq-appliance/lab8/image127.png)
    
	Click **Cancel** to dismiss the dialog.
	
	HAQM2 is now running and is now the DR Primary.

6. Add a widget for **Queues on HAQM2** on the **MQAppl3** MQ Console, and
    position it next to the *Local Queue Managers* widget.

    ![](./images/pots/mq-appliance/lab8/image128.png)
    
7. Oh look! There are three messages on the **TEST2** queue. This is
    expected since you gave TEST2 default persistence.

    ![](./images/pots/mq-appliance/lab8/image129.png)

8. Browse those message, and as you would expect they are the messages
    you put on the queue while HAQM2 was running on MQAppl2. Click
    **Close** to dismiss the display.

    ![](./images/pots/mq-appliance/lab8/image130.png)
    
    ![](./images/pots/mq-appliance/lab8/image131.png)
    
	HAQM2 has successfully failed over to the disaster recovery site and all
messages are available for processing. Client applications will need to
reconnect to 10.0.0.3, and they would connect automatically if the CCDT
has both addresses coded.

Congratulations, you have completed Lab 8 Disaster Recovery.
