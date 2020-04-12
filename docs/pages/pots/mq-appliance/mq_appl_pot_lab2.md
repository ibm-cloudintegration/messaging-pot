---
title: IBM MQ Appliance High Availability
toc: false
sidebar: labs_sidebar
folder: pots/mq-appliance
permalink: /mq_appl_pot_lab2.html
summary: High Availability 
applies_to: [developer,administrator]
---

# Lab 2 - IBM MQ Appliance High Availability


In this lab, you will configure two virtual appliances for high
availability (HA) and test that HA works as expected.

The lab environment consists of two virtual appliances (MQAppl1 and
MQAppl2) and a Windows environment to perform console operations and
testing. There are two other virtual appliances (MQAppl3 and MQAppl4)
that will not be used in this lab. You should suspend them.

The virtual appliances you will use for this lab will be **MQAppl1**,
**MQAppl2** and **Windows 10 x64** in the CSIDE environment.

If you successfully completed Lab 1 on a CSIDE environment, you can
continue to use your CSIDE environment. If so, you must have configured
MQAppl2 in addition to MQAppl1. Otherwise, you can use the CSIDE
template **MQ Appliance PoT 9.1.4 Configured -- ready for HA**, which is
the solution for Lab 1. 

{% include important.html content="**This lab assumes that Lab 1 has been completed! 
You must either complete Lab 1 before attempting this lab, or use the “MQ Appliance PoT 9.1.4 Configured – Ready for HA” CSIDE template.** 

If Lab 1 has not been completed on the virtual appliance you are using, the results you see will differ from the examples in this lab guide." %} 

## Start the environment

1. Wait for the virtual machines to power on. MQAppl1 is shown below.
    MQAppl2 should appear the same.
    
    ![](./images/pots/mq-appliance/lab2/image9.png)

2. Log in with user / password **admin** / **passw0rd**.

	![](./images/pots/mq-appliance/lab2/image10.png)
	
	![](./images/pots/mq-appliance/lab2/image11.png)

	{% include note.html content="If you see a message that states *Notice: startup config contains errors*, you can ignore the message. " %}

## The virtual environment

### Virtual appliance MQAppl1 

1. The first virtual appliance you will look at is MQAppl1. Check that
    this appliance is in a running state.

2. Make sure you are at the appliance command line. If you are in the
    mqcli, type "**exit**". Execute the "**show ipaddress**" command. The IP
    addresses in use for this appliance are as follows:

    ![](./images/pots/mq-appliance/lab2/image12.png)
    
    You need to be at the *mqcli* command line to issue the mq commands. 
    
3. Enter **mqcli**

	![](./images/pots/mq-appliance/lab2/image12a.png)      
    
### Virtual appliance MQAppl2

1. The second virtual appliance you will look at is MQAppl2. Check that this appliance is in a running state and log on with the same credentials as MQAppl1 -- **admin** / **passw0rd**.

2. Make sure you are at the appliance command line. If you are in the mqcli, type "**exit**". Execute the "**show ipaddress**" command. The IP addresses in use for this appliance are as follows:

    ![](./images/pots/mq-appliance/lab2/image13.png)
    
3. Enter **mqcli**

## Create the HA Group

You will now create the HA group on the two appliances. You should be at the mqcli command line, as shown below:

1. On *MQAppl2*, run the following command:

	**`prepareha -s SomeSecret -a 10.0.1.1`**

	![](./images/pots/mq-appliance/lab2/image15.png)
	
	{% include note.html content="About *prepareha*:  
	This command prepares an appliance to be part of an HA group. You run it on the appliance that you do not run *crthagrp* on.
	
	 '*-a 10.0.1.1*' 
	Specifies the IP address on the HA group primary interface, on the other appliance in the group.
	
	'*-s SomeSecret*'
	Specifies a string that is used to generate a short-lived password. The password is used to set up the unique key for the two appliances.   "%}     
	
2. Now go to *MQAppl1* (do not wait for the prepareha command to
    complete) and issue the following command:

	**`crthagrp -s SomeSecret -a 10.0.1.2`**

	![](./images/pots/mq-appliance/lab2/image16.png)
	
	{% include note.html content="About *crthagrp*: 
	This command creates an HA group of two appliances. The prepareha command must be run on the other appliance before you run *crthagrp*. 
	
	'*-a 10.0.1.2*'
	Specifies the IP address on the HA group primary interface, on the other appliance in the group.
	
	'*-s SomeSecret*'
	Specifies a string that is used to generate a short-lived password in the preparha command. It is the unique key for the two appliances. "%} 

3. This may take a few minutes. At the completion of the command execution, you should see the following messages:

    ![](./images/pots/mq-appliance/lab2/image17.png)
    
4. Check the results on *MQAppl2*. You will see that the HA group was
    successfully created.

    ![](./images/pots/mq-appliance/lab2/image18.png)

	You are now ready to create queue managers and test the HA.

## Create queue managers 

1. On the *MQAppl1* appliance, issue the following command:

	**`crtmqm -p 1511 -fs 2 -sx HAQM1`**


	{% include note.html content="About filesystem size (-fs) and (-sx):

	The -fs parameter specifies that the queue manager is created with the file system size fs. 
	
	The default file system size is set to 64GB on a real appliance, but you set it to only 2GB on the virtual appliance. 
	
	The -sx parameter specifies that the queue manager is a high availability (HA) queue manager. " %}

2. After the queue manager has been created, you will see the HA
    configuration taking place as shown below. You should see the
    message indicating that the final HA configuration has succeeded.

	![](./images/pots/mq-appliance/lab2/image19.png)

	{% include note.html content="About high availability status:  
	
	You can view the status of a queue manager in a high availability (HA) group by using the status command on the command line.  

	The status command returns information about the operational status of a specified queue manager in the HA group. The status can include the following information:
	the operational state of the HA group,
	the filesystem size and CPU used by the queue manager,
	the replication status of the queue manager (if synchronization is in progress), the preferred appliance for the queue manager,
	whether a partitioned situation is detected and if it has, the amount of 'out-of-sync' data held. 
	" %}

3. You should now run the following command to check the status of the
    queue manager:

	**`status HAQM1`**

4. You should now see that the queue manager is running, HA is enabled
    and running normally, with "This appliance" as the preferred
    location.

	![](./images/pots/mq-appliance/lab2/image20.png)

5. If you do not see output as above, try running the command again as
    synchronization may still be in progress.

6. Now, go to the *MQAppl2* appliance.

7. Create another HA queue manager. Issue the following command:

	**`crtmqm -p 1512 -fs 2 -sx HAQM2`**

8. Again, you expect to see the successful creation of the queue
    manager and successful completion of the HA configuration.

	![](./images/pots/mq-appliance/lab2/image21.png)
	
9. Run the status command for this queue manager to check that initial
    synchronization has completed successfully.

	**`status HAQM2`**

10. Staying on *MQAppl2*, run the status command for the HAQM1 queue
    manager. If you contrast this with the HAQM2 status results you see
    the following:

    ![](./images/pots/mq-appliance/lab2/image22.png)

	The status shows us that the HAQM1 queue manager is running on
    another appliance, and that that other appliance is the preferred
    location for HAQM1.

	You are now ready to start testing HA, but first you need to set up the
MQ Explorer.

## Set up MQ Explorer	

We will use the *testuser* messaging user that was created in Lab 1. 

{% include note.html content="Note:  This
is different from the user who administers the appliance itself." %}

1. Validate that *testuser* is defined, by entering the following command on either appliance:

	**`userlist -u testuser`**

	![](./images/pots/mq-appliance/lab2/image22a.png)

	If *testuser* does not exist, enter the following command on both MQAppl1 and MQAppl2:
	
	**`usercreate -u testuser -p passw0rd -g mqm`**
	
	You now need to set up the **SYSTEM.ADMIN.SVRCONN** channel that the MQ Explorer uses for communication.

2. Go to the MQAppl1 appliance and enter the following commands:

	~~~~
	runmqsc HAQM1
	
	DEFINE CHANNEL(SYSTEM.ADMIN.SVRCONN) CHLTYPE(SVRCONN)
	
	SET CHLAUTH(SYSTEM.ADMIN.SVRCONN) TYPE(BLOCKUSER) USERLIST('*whatever')
	
	ALTER AUTHINFO('SYSTEM.DEFAULT.AUTHINFO.IDPWOS') AUTHTYPE(IDPWOS) ADOPTCTX(YES)
	
	REFRESH SECURITY TYPE(CONNAUTH)
	
	END
	~~~~

	![](./images/pots/mq-appliance/lab2/image24.png)

3. Go to the *MQAppl2* appliance and repeat all of the steps in this Set up
    MQ Explorer section (replacing HAQM2 for HAQM1).

	You are now ready to add the appliance HA queue managers to MQ Explorer.

4. Open the VM (Windows 10 x64) where the MQ Explorer and browser
    reside.

5. Log on as **ibmdemo** (password = passw0rd)

6. Open **MQ Explorer** (from the desktop).

    ![](./images/pots/mq-appliance/lab2/image25.png)

7. Right-click the **Queue Managers** folder and select **Add Remote
    Queue Manager...**

	![](./images/pots/mq-appliance/lab2/image26.png)

8. Enter the name of the MQAppl1 queue manager (**HAQM1**) and select
    **Connect directly**.

	![](./images/pots/mq-appliance/lab2/image27.png)

9. Click **Next**.

10. Enter the IP address of the MQAppl1 appliance (**10.0.0.1**) and the
    port number of the listener (**1511**).

    ![](./images/pots/mq-appliance/lab2/image28.png)

11. Click **Next** twice.

12. Click **Enable user identification**.

13. Enter the messaging user (**testuser**) and click the **Use saved
    password** radio button.

    ![](./images/pots/mq-appliance/lab2/image29.png)

14. Click **Enter password** and enter the password for the messaging
    user (**passw0rd**).

    ![](./images/pots/mq-appliance/lab2/image30.png)

15. Click **OK**, then click **Finish**.

16. Repeat the steps above to add the
    HAQM2 using the following details:

	-   Queue manager (**HAQM2**)

	-   IP address (**10.0.0.2**)

	-   Listener port (**1512**)

	-   Messaging user and password (**testuser** / **passw0rd**)

17. You will now see the two MQ Appliance queue managers in the Queue
    Managers folder.

    ![](./images/pots/mq-appliance/lab2/image31.png)

18. In the content pane, you will see that the queue managers are
    identified as Appliance queue managers.

	![](./images/pots/mq-appliance/lab2/image32.png)

You are now ready to test the HA Failover.

## Test HA Failover

1. Go back to **MQAppl1**.

2. Ensure you are at the mqcli interface and issue the following command:

	**`sethagrp -s`**

	{% include note.html content="About *sethagrp -s*. 
 This command pauses and resumes an appliance in a high availability group. When you use the sethagrp command to pause (or suspend) an appliance that is part of a high availability group, any queue managers running on that appliance fail over to the other appliance in the group.  "%} 
                  

3. Run a **status HAQM1** command and note what it displays.

    ![](./images/pots/mq-appliance/lab2/image33.png)

4. Go back to the MQ Explorer and add the HAQM1, but this time use the
    following details to add it as if it was running on MQAppl2 rather
    than MQAppl1

	-   Queue manager (**HAQM1**]

	-   IP address (**_10.0.0.2_**)

	-   Listener port (**_1511_**)

	-   Messaging user and password (**testuser** / **passw0rd**)

5. You will now see three queue managers listed, but both queue
    managers are now running on the MQAppl2 appliance.

    ![](./images/pots/mq-appliance/lab2/image34.png)

6. The queue manager on MQAppl1 appliance shows as disconnected and
    does not show a status.

    ![](./images/pots/mq-appliance/lab2/image35.png)

	Now resume the appliance from standby mode.

7. Go back to *MQAppl1* and enter the following command:

	**`sethagrp -r`**

8. Check the status of the HAQM1 queue manager and note what it
    displays.

    ![](./images/pots/mq-appliance/lab2/image36.png)

9. If the queue manager is not yet showing as *Running*, it is still in
    the process of failing back. Run the status command again.

	Is it running on MQAppl1 again? If it is, you can proceed to the
    next step.

	![](./images/pots/mq-appliance/lab2/image37.png)

10. Go to the MQ Explorer and check that it is also showing the HAQM1 as
    running on the MQAppl1 appliance. Note: if not, you may have to
    reconnect to the queue manager (*right-click* on the queue manager and select *Connect*).

	Do the same failover test, but for the **HAQM2** queue manager on the
**MQAppl2** appliance.

11. Go back to the MQAppl2 appliance and run the sethagrp command to
    suspend (**sethagrp -s**) as before.

12. Check the status of the HAQM2 queue manager to verify that the
    status indicates that it is running elsewhere.

13. Check in the MQ Explorer to verify that you can add (and then see)
    the HAQM2 running on the MQAppl1 (10.0.0.1) appliance.

    ![](./images/pots/mq-appliance/lab2/image38.png)

14. Finally, go back to the MQAppl2 and use the **sethagrp -r** command
    to resume the appliance.

15. Verify that both queue managers are running on the appropriate
    appliance.

Finally, for this lab you need to process some messages in the HA
environment.

## <a name="Process_messages"></a>Process Messages

In this section, you will create queues and process messages to and from
these queues. You may use any combination of your favorite tools to do
any of this (MQ Explorer, rfhutil, any of the sample programs for MQ).
However, start with the Web console because it is a very useful
interface to the appliance queue managers.

1. Open a **Firefox** browser on Windows.

2. You want to have two tabs in the browser, one for the web console of each appliance.

3. If you receive any exception messages when opening the console URLs,
    add an exception and continue.

4. Select the first tab and log on to the console for *MQAppl1* (**user =
    admin / password = passw0rd**).

5. Click **MQ Console**.
    
    ![](./images/pots/mq-appliance/lab2/image39.png)

6. First, notice the **High Availability** status at the top of the
    console window, showing a checkmark to signify that the high
    availability status of the HA group is good -- that both appliances
    are up and running part of the HA group. Clicking **High
    Availability** gives you a menu where you can suspend or resume this
    appliance, see the HA queue manager status, or delete the HA group.
    If you have not created an HA group, this option instead allows you
    to create the HA group.

	The next interesting thing you see is the queue manager status.

    ![](./images/pots/mq-appliance/lab2/image40.png)

	You see both queue managers in a running and highly available state.

	You will now add a widget for the queues.

7. Click the '**+**' symbol and add a new tab called **HAQM1**. Click **Add widget** to add a widget to the layout.

	![](./images/pots/mq-appliance/lab2/image41.png)

8. In the *Add a new widget* popup, click **Local Queue Managers**.

	![](./images/pots/mq-appliance/lab2/image41a.png)

9. Now click on **HAQM1** in the Local Queue Manages widget to select it, and
    again click **Add widget**. 

	The **Queue Manager** field is pre-selected because you selected the
    queue manager from the Local Queue Managers widget. If you want to
    change to another queue manager, use the drop-down to select the
    desired one.

10. Click **Queues** from the list, to see the queue objects for HAQM1.

    ![](./images/pots/mq-appliance/lab2/image42.png)

11. The widget now appears in the dashboard in another pane entitled
    "Queues on HAQM1".

    ![](./images/pots/mq-appliance/lab2/image43.png)

	You now need to create the queue you will use for testing.

12. In the **Queues** widget, click **Create +**.

    ![](./images/pots/mq-appliance/lab2/image44.png)

13. Name the queue **Q1** and leave the default object type set to
    local. Click **Create**.

    ![](./images/pots/mq-appliance/lab2/image45.png)

14. Click **Q1**, and then click the **Properties** icon.

    ![](./images/pots/mq-appliance/lab2/image46.png)

15. In the Properties pop-up box, set the properties for the new queue. Click the
    drop down box for **Default persistence** and set it to
    **Persistent**. Then click **Save**.

    ![](./images/pots/mq-appliance/lab2/image47.png)
    
    Notice that it informed you of unsaved changes before you saved, and then it informed you it saved the changes after you clicked save.
    
16. Click **Close** to close the Properties dialog.

	You now need to put some messages into the Q1 queue. Use the MQ
    Explorer for this, because you can easily do it without having to
    perform any additional configuration.

17. Open the MQ Explorer content pane for the HAQM1 queues.

18. Right-click **Q1** and select **Put Test Message...**

    ![](./images/pots/mq-appliance/lab2/image48.png)

19. Enter some test messages by entering some text and clicking **Put
    message.**

	![](./images/pots/mq-appliance/lab2/image49.png)

20. Repeat this for as many messages as you wish to put on the queue
    (the lab test scenario has 13 messages).

21. When you are finished, click **Close**.

22. Go back to the Web console dashboard and refresh the display by
    clicking the ![](./images/pots/mq-appliance/lab2/image50.png) *Refresh widget* symbol.

    ![](./images/pots/mq-appliance/lab2/image51.png)

23. You should now see that there are 13 messages on the queue (or as
    many messages as you put there).

    ![](./images/pots/mq-appliance/lab2/image52.png)

	You now need to test HA to ensure that the messages fail over to
    the queue and queue manager on the other appliance.

24. Go to the *MQAppl1* appliance.

25. Suspend the appliance using the **sethagrp -s** command as before.

26. Use the **dspmq** command to verify that the HAQM1 is running
    elsewhere (as before, if this takes a little time, continue to run
    the command until the results are as shown below).

    ![](./images/pots/mq-appliance/lab2/image53.png)

27. Go back to the browser, but this time log on to the Web console for
    the MQAppl2 appliance (this is the other tab in the browser).

28. As before, add an MQ Object Widget for queues to the dashboard.
    Select **HAQM1**

    ![](./images/pots/mq-appliance/lab2/image54.png)

29. This time, we want to give the widget a different name so click the
    widget name edit icon.

    ![](./images/pots/mq-appliance/lab2/image55.png)

30. Name it "**Failed over HAQM1**" to indicate that you are looking at
    the failed over queue manager. Click **Rename**.

    ![](./images/pots/mq-appliance/lab2/image56.png)

31. As you can see, the 13 messages are all present on Q1 (on *HAQM1* on *MQAppl2*).

    ![](./images/pots/mq-appliance/lab2/image57.png)

	From here, you can browse the messages on the queue.

32. Click on the **Q1** queue, and then click the
    ![](./images/pots/mq-appliance/lab2/image58.png) **Browse** icon.

    ![](./images/pots/mq-appliance/lab2/image59.png)

33. Verify that the messages all appear as you expect.

34. Click **Close**.

    ![](./images/pots/mq-appliance/lab2/image60.png)

	You now want to put the queue managers back the way they were.

35. Go to the *MQAppl1* appliance.

36. Issue the **sethagrp -r** command to resume the appliance.

37. Go back to the Web console dashboard for MQAppl1. Refresh the **Queue Managers** widget, if necessary, to see that HAQM1 is now
    running on this appliance again and HAQM2 is still running on the
    other appliance.

38. Also, refresh the **Queues** widget to see that the 13 messages for
    Q1 are back where they belong, on the queue belonging to this queue
    manager on this appliance.

    ![](./images/pots/mq-appliance/lab2/image62.png)
    

You have now successfully completed the setup and testing of the HA
environment between two MQ Appliances. This officially ends this lab. If
time allows, you may continue with any of two extra credit sections: 

[Run sample client applications](#Extra_credit)

[See how to manage HA with the MQ Console](#Configure_HA_using_MQ_Console)


## <a name="Extra_credit"></a>Extra Credit

If you have time to spare and want to try more testing, you may wish to
test failing over the other queue manager. Alternatively, you may wish
to test using other sample MQ utilities such as amqsputc and amqsgetc.

Bear in mind that if you want to run these utilities from the browser /
MQ Explorer image you will also need to do additional set up.

### Set up SYSTEM.DEF.SVRCONN

You can use the default svrconn channel for your client
    communication with the queue managers, but you will need to
    configure the channel authentication as you did previously for the
    SYSTEM.ADMIN.SVRCONN.

1. Using **runmqsc** on each of the appliances, perform the following:

	~~~~
	SET CHLAUTH(SYSTEM.DEF.SVRCONN) TYPE(BLOCKUSER)
USERLIST('*whatever')

	ALTER AUTHINFO('SYSTEM.DEFAULT.AUTHINFO.IDPWOS') AUTHTYPE(IDPWOS) ADOPTCTX(YES)

	REFRESH SECURITY TYPE(CONNAUTH)

	END
	~~~~

### Set up variables

You also need to set the connection information for the MQSERVER and
MQSAMP\_USER\_ID variables.

Depending on whether you are running your test in a Windows or a
    Linux environment, they will have a slightly different format.

1. For Windows:
	
	~~~~
	set MQSERVER=SYSTEM.DEF.SVRCONN/TCP/ipaddress(port) 

	set MQSAMP_USER_ID=testuser
	~~~~

2. For Linux:

	~~~~
	export MQSERVER=SYSTEM.DEF.SVRCONN/TCP/'ipaddress(port)'

	export MQSAMP\_USER\_ID=testuser
	~~~~

You can change these variables to suit whichever particular appliance
and queue manager you are running a test for.

If you are unfamiliar with the sample programs and wish to use them,
please speak to the instructor.

## <a name="Configure_HA_using_MQ_Console"></a>Configure HA Using MQ Console


This section is included to **show** how to create an HA group and HA queue
managers using the MQ Console.

1. Open Firefox. Click the *MQAppl2* bookmark.

2. Sign in with **admin** / **passw0rd**.

3. Click **High Availability** and select **Create new group...**.

	![](./images/pots/mq-appliance/lab2/image63.png)

4. Click **First**.

    Enter the IP address for the MQAppl1 HA interface **10.0.1.1**

    Enter "**SomeSecret**" in the Secret text field

    Click **Prepare**

    ![](./images/pots/mq-appliance/lab2/image64.png)

5. This message means the HA group is being prepared and you have
    additional work to do.

    ![](./images/pots/mq-appliance/lab2/image65.png)

6. Open a new browser tab and click the *MQAppl1* bookmark.

    Click **High Availability** and select **Create new group...**.

    Click **Second** and enter the IP address of the HA interface for
    MQAppl2 -- **10.0.1.2** and the secret text you entered on MQAppl2.

    Click **Connect**

    ![](./images/pots/mq-appliance/lab2/image67.png)

7. The HA group will now get created.

    When the creation of the HA group is complete, you will see a
    checkmark for High Availability.
    
    ![](./images/pots/mq-appliance/lab2/image67a.png)

8. Click **High Availability** and you will see that both appliances
    are online for the HA group.

    ![](./images/pots/mq-appliance/lab2/image68.png)

9. Return to the *MQAppl2* browser and make sure High Availability
    is enabled.

    ![](./images/pots/mq-appliance/lab2/image69.png)
    
	Notice that no queue managers have been created yet. You must have
    the HA group defined before creating queue managers in the HA group.

10. Click **High Availability** and select **HA queue manager status...**.
    Click **Cancel** to dismiss the status window.

    ![](./images/pots/mq-appliance/lab2/image70.png)
    
11. In the *MQAppl1* browser, click the **Create +** button in the **Local Queue Managers** widget to add a new HA
    queue manager.

    Fill in the panel with the following values:

	-   Queue manager name: **HAQM1**

	-   Port: **1511**

	-   File system size: **2 GB**

	-	Startup: **Automatic**

	Click **Next**.

    ![](./images/pots/mq-appliance/lab2/image71.png)     

12. On the second page, click **Replicated** (meaning it is a highly available queue manager, instead of a standard non-HA queue manager). Now click **Create**.

    ![](./images/pots/mq-appliance/lab2/image72.png)

	Notice that you have the opportunity right then to define a floating IP address for the HA queue manager.

13. The queue manager gets created and replicated to the second appliance. This will take a few minutes.

    ![](./images/pots/mq-appliance/lab2/image73.png)

14. Once the creation and replication is completed, it will change to a running status.

    ![](./images/pots/mq-appliance/lab2/image74.png)

15. Switch to the *MQAppl2* browser. You will see HAQM1 in the **Local Queue Managers** widget, but this time
    it shows running on the other appliance.

    ![](./images/pots/mq-appliance/lab2/image76.png)

16. Now create another queue manager on *MQAppl2* with the following values:

	-   Queue manager name: **HAQM2**

	-   Port: **1512**

	-   File system size: **2 GB**

	-	Startup: **Automatic**

	-	High Availability: **Replicated**

	Again, the queue manager will be created and replicated to the second appliance. 

17. Creation complete -- running queue manager. HAQM1 running on
    MQAppl1 (elsewhere), and HAQM2 running here.

    ![](./images/pots/mq-appliance/lab2/image80.png)

18. Switch to the *MQAppl1* browser. Ensure high availability is
    enabled. HAQM1 running locally (MQAppl1), HAQM2 running on MQAppl2
    (elsewhere).

    ![](./images/pots/mq-appliance/lab2/image81.png)

19. Staying on MQAppl1, click **High Availability** and select
    **Suspend this appliance...**.

    ![](./images/pots/mq-appliance/lab2/image82.png)

20. You get a warning. Click **Suspend** to confirm.

    ![](./images/pots/mq-appliance/lab2/image83.png)

21. The work to suspend the appliance starts. When complete, an alert is posted and HAQM1 will show as *Running elsewhere*. You will also notice the symbol now next to the *High Availability" button.

    ![](./images/pots/mq-appliance/lab2/image84.png)
    
    {% include note.html content="Notice that 'suspending' the appliance in the HA group simply temporarily suspends its participation in the HA group. The appliance is still running, just not doing any HA replication, allowing you to now perform actions such as a firmware upgrade on the suspended appliance.
	" %}

22. Click **High Availability** again and you will see what the alert
    is all about. Notice that this appliance is in *Standby*.

    ![](./images/pots/mq-appliance/lab2/image85.png)    

23. Return to the *MQAppl2* browser and see that the queue manager
    statuses have been updated automatically and an HA alert is posted.

    ![](./images/pots/mq-appliance/lab2/image86.png)

24. Back on *MQAppl1*, click **High Availability** once more and
    select **Resume this appliance...** to bring MQAppl1 back online in the
    HA group.

    ![](./images/pots/mq-appliance/lab2/image87.png)

25. Repeat the process by suspending MQAppl2 and observe the queue
    manager statuses on both appliances.

125. Return to [Process Messages](#Process_messages)
 to process messages on HA queue managers.

Congratulations, this concludes the HA lab.
