---
title: Publish / Subscribe Administration
toc: false
sidebar: labs_sidebar
folder: pots/mq-basic
permalink: /mq_basic_pot_lab3.html
summary: Managing the MQ Pub / Sub Environment
applies_to: [administrator]
---

# Lab 3 - Publish / Subscribe Administration 

## Lab Overview
This lab will demonstrate the administration of Publish / Subscribe-related
objects using the IBM MQ Explorer. Also, you will use the MQ Explorer Test Publication and Test Subscription features to demonstrate publishing and
subscribing to TOPIC strings.

You will be working with the hierarchical structure of TOPIC STRINGS represented in the picture below.

![](./images/pots/mq/lab3/image1.png)

[View an overview of MQ PubSub Admin](https://ibm.box.com/s/hjyg159jzclk4v8xgl9cwpi0zmy83kem)

## Using MQ Explorer to create and display information 
You may use the same demo environment that you used for Lab 1. If you are doing this lab consecutively following Lab 1, you can continue with that environment. If you are doing this lab independent of Lab 1, you may need to create a new demo environment. In that case click the link below.

[See the Environment Setup section](https://pages.github.ibm.com/cloudintegration/PoT-messaging/env_setup.html)    

 {% include note.html content="The following instructions assume that you are using the same demo environment from Lab 1. In this case you have created the **MQPOT** queue managers. If the MQPOT queue manager does not appear in the MQ Explorer, you must create it according to the instructions in Lab 1." %}
 

1. If the MQ Explorer is not already running, you can launch it from
    the shortcut on the desktop.

    ![](./images/pots/mq/lab3/image2.png)

2.  Start the IBM MQ Explorer by double-clicking the **MQ
    Explorer** shortcut.

3.  Click the **Topics** folder in the Navigator pane (left side). Notice that you have no topic objects defined yet! We have provided a script for this purpose.

    ![](./images/pots/mq/lab3/image3.png)

4.  To run the provided script, find the shortcut called **PubSub Lab Setup** on the Windows 10 desktop. Double-click the shortcut to run the script.

    ![](./images/pots/mq/lab3/image4.png)

    If you receive an Open File – Security Warning popup, click **Run** to allownthe script to execute.

    ![](./images/pots/mq/lab3/image5.png)

5.  The script should run very quickly, leaving the following command window
    open. Verify that the command was successful. Then press Enter to close the command window.

    ![](./images/pots/mq/lab3/image6.png)

6.  You should now see the following MQ Topic objects displayed in the Content pane in the MQ Explorer. Observe the Topic *objects* (under the Topic name column) and their corresponding Topic *string* values. Also observe the Publish and Subscribe enablement status on the right.

    ![](./images/pots/mq/lab3/image7.png)

7.  Double-click the **SPORT** Topic object.

    ![](./images/pots/mq/lab3/image8.png)

8.  Observe the various properties of the Topic object. Explore the various values available on the pull-downs if you wish. Close the window by clicking **Cancel**.

    ![](./images/pots/mq/lab3/image9.png)

9.  From the Navigation pane select **Topics** under queue manager MQPOT. Right-click **Topics** and then select **New Topic** from the context menu.

    ![](./images/pots/mq/lab3/image10.png)

10. Enter **MONEY** in the Name field; allow the other fields to default. Then click “**Next**”.

    ![](./images/pots/mq/lab3/image11.png)

11. Enter **“finance/cash/gettingit/frombanks**” (without the quote marks) in the **Topic string** field. Enter a description in the description field and then click “**Finish**”

    ![](./images/pots/mq/lab3/image12.png)

12. Close the confirmation box by clicking **OK**.

    ![](./images/pots/mq/lab3/image13.png)

13. Back in the MQ Explorer, right click **Topics** (under MQPOT) and then select **Status**.

    ![](./images/pots/mq/lab3/image14.png)

14. Now click the “+” symbol to the left of **finance** – a level of the hierarchy opens; repeat on the “+” in front of **cash**, then **gettingit**, then **frombanks** which is the bottom of the “*tree*”.

    In the Topic Status notice that all the intermediate nodes have been created and that they have inherited properties from the parent **finance**. These intermediate nodes have no related Topic Objects and so cannot have their properties altered by MQSC or MQ Explorer.

15. Close the Topic Status view by clicking **Close**.

    ![](./images/pots/mq/lab3/image15.png)

***This concludes this portion of Lab 3.***

## A First look at the MQ Explorer Pub/Sub Test tools
In this section you will be using the tools that come with IBM MQ Explorer that allow you to test publishing to and subscribing to topics.

2.  You will now be working with some pre-defined topics. From the MQ Explorer, display topic status by right-clicking the **Topics** folder and selecting **Status…**

    ![](./images/pots/mq/lab3/image16.png)

3.  You are going to focus on the **sport** topic tree. Expand the “+” symbols on the **sport** topic tree and you should see something similar to the screen capture below. Notice that Publish is *allowed* for the topic string **sport/football/results/hursley**. Also, following the tree “up”, you should notice that the topic string **sport/football** has the publish attribute **Inhibited.** *Remember this* as you complete the next steps of this lab. Click **Close** to dismiss the status window.

    ![](./images/pots/mq/lab3/image17.png)

4.  Start a test subscription window by right-clicking **Topics** and selecting **Test Subscription.**

    ![](./images/pots/mq/lab3/image18.png)

5.  Type in the topic string **sport/football/\#** and press the **Subscribe** button. 
    The “\#” symbol is called the *multi-level wildcard*. The string **sport/football/\#** indicates a subscription to all publications sent to the sport/football topic or any of its children. The Test Tool window remains open and the **Unsubscribe** button becomes active. Publications received will be displayed in the **Messages received** box.

    ![](./images/pots/mq/lab3/image19.png)

6.  **Minimize** the Subscribe window by clicking the minimize button.

    ![](./images/pots/mq/lab3/image20.png)

7.  The Subscribe window will “park” itself at the bottom left part of the Windows desktop. You will restore this window in a later step of this lab.

    ![](./images/pots/mq/lab3/image21.png)

8.  Right click the Topics folder then select **Status…**

    ![](./images/pots/mq/lab3/image22.png)

9.  Expand the **sport** tree. Observe the Subscription counts; sport/football and its children have a positive subscription count. You will need to scroll to the right to find the **Sub count** column. Close the topic status window.

    ![](./images/pots/mq/lab3/image23.png)

10. From the Topic display, select the **SPORT.FOOTBALL** row, right-click and select **Topic Status – Subscribers**. This gives detailed information about subscribers to this Topic object.

    ![](./images/pots/mq/lab3/image24.png)

11. Observe the detailed display and then close the status screen by clicking the **Close** button.

    ![](./images/pots/mq/lab3/image25.png)

12. Now you will *publish* a message. Returning to the Topic list, select the **SPORT.FOOTBALL** row, right-click and then select **Test Publication**.

    ![](./images/pots/mq/lab3/image26.png)

13. This dialog will publish a message to the topic string **sport football**.

    ![](./images/pots/mq/lab3/image27.png)

14. Before entering a message and sending it, you will arrange the windows on the screen.

15. Locate the **Subscribe** Test Tool window where you previously subscribed to **sport/football/\#**. Click the Restore window button to restore the window. 

    ![](./images/pots/mq/lab3/image28.png)

16. Now position the **Publish Test Message** and the restored **Subscribe** test tool windows so they both are visible. Then return focus to the **Publish** window.

    ![](./images/pots/mq/lab3/image28a.png)

16. Type a message such as **Hello World** and then press **Publish Message**. 

    ![](./images/pots/mq/lab3/image29.png)

17. An error occurs and a pop-up shows you the return code 2051. You can click *Details* but that does not provide any further explanation.

    ![](./images/pots/mq/lab3/image29a.png)

1. Open a command prompt, type *mqrc 2051* and hit enter. You can see that 2051 indicates that put is inhibited. 

    ![](./images/pots/mq/lab3/image29b.png)
    
    This because the topic object for **sport/football** is **publish-inhibited**. You’ll recall that we saw that this was set earlier in the lab. But this will not inhibit our subscribers; we used the multi-level wildcard to subscribe to topics at and below “sport/football” in the topic tree, so we will be subscribing to items published lower in the hierarchy.

18. Click Close to dismiss the error popup.

19. In the Publish Test Message window, overtype the topic string to **sport/football/news/hursley** and click **Publish message**. You have published and subscribed your first message!

    ![](./images/pots/mq/lab3/image31.png)
    
    {% include note.html content="Dynamically Created Topic Objects. These dynamically created Topic objects are temporary and only exist for a limited amount of time before the Queue Manager removes them; for example if you restart the Queue Manager, they will no longer exist.." %}   

1.  Now try publishing to **sport/football/news/hursley/fundraising/raffle**. Type a new message such as "Hello World - raffle" in the publish window. The message is sent to the subscriber. New levels of the hierarchy have been created automatically.

    ![](./images/pots/mq/lab3/image32.png)

2.  Now try publishing to **sport/football/rules/offside**. The publish attempt failed! That is because the node in the topic tree that is dynamically created automatically inherits the properties of the parent **sport/football** – which has its Publish attribute Inhibited. Click **Close** to close the error popup.

    ![](./images/pots/mq/lab3/image33.png)

3.  Return to the Topic object display. Once again open the Topic status list and expand the **sport** hierarchy. You will see the automatically created elements. Click **Close** to close the status window.

    ![](./images/pots/mq/lab3/image34.png)

    ***This concludes this portion of Lab 3.***

## Administered Subscriptions
While it is typical for subscribers to register their own subscriptions, it is possible to administratively register a subscription using MQ
Explorer. This is a subscription to a topic string that delivers messages to a queue. This can be very useful because it is a way for a legacy program which was designed as a point-to-point application to read a queue associated with a topic; in this way it can participate in publish/subscribe without changing the program. You will now explore how such a subscription can be set up and used.

1.  In the MQ Explorer, select **Queues**. Right-click and select **New Local Queue**.

    ![](./images/pots/mq/lab3/image35.png)

2.  Name the queue **ALL_FOOTBALL_Q** and press **Finish**.

    ![](./images/pots/mq/lab3/image36.png)

3.  Click **OK** to close the confirmation dialog.

    ![](./images/pots/mq/lab3/image37.png)

4.  Select **Subscriptions**, right-click and select **New Subscription**.

    ![](./images/pots/mq/lab3/image38.png)

5.  Type **ALL_FOOTBALL_SUB** as the subscription name and then click **Next**.

    ![](./images/pots/mq/lab3/image39.png)

6.  Leave the Topic Name blank, and enter **sport/football/\#** as the Topic string. Leave the Destination Queue Manager blank and enter **ALL_FOOTBALL_Q** in the Destination Name. Then click **Finish**.

    ![](./images/pots/mq/lab3/image40.png)

7.  Click **OK** to close the confirmation window.

    ![](./images/pots/mq/lab3/image41.png)

8.  The new administrative subscription appears.

    ![](./images/pots/mq/lab3/image42.png)

    Double-click the new subscription to see its attributes.

9.  The attributes of the new subscription are displayed.

    ![](./images/pots/mq/lab3/image43.png)

    This subscription will now route all qualifying messages to the local queue **ALL_FOOTBALL_Q**.

10. Close the properties window by clicking **Cancel**.

***This concludes this portion of Lab 3.***

## Testing Publications and Subscriptions from the command line

You will now use two more sample programs that are supplied with IBM MQ to further test IBM MQ publish and subscribe capabilities, called amqspub and amqssub.

1.  A folder on the desktop contains four shortcuts that will start two instances of a publishing sample and two subscribers. Open the folder and then double-click each shortcut to launch the programs.

    ![](./images/pots/mq/lab3/image44.png)

    ![](./images/pots/mq/lab3/image45.png)

    If you receive an **Open File – Security Warning**, click **Run** or **Yes** to allow the program to run.

    ![](./images/pots/mq/lab3/image46.png)

2.  Arrange the windows as shown, so that you can see all of them at them same time. The top two windows are the *topic publishers* (amqspub) Each time you type text into either window, the windows on the bottom, the *topic subscribers* (amqssub) will receive the text as published messages because the topic string that they are subscribing to matched the one being used by the publishers.
    
    {% include note.html content="Subscriber Timeout. The amqssub program times out after 30 seconds if no messages arrive. If your window times out, just double-click the command again to restart the program..." %}

    ![](./images/pots/mq/lab3/image47.png)

1.  Now in the top *left* window (publishing to **sport/football/news/hursley**) enter “**test message 1**” and press Enter. The message should appear in *both* subscribing windows because the published message matched both subscriptions.

    Again in the top *left* window type the text **Hursley News** and then press **Enter**.

    In the top *right* window type the text **Football News** and press **Enter**. Notice that the **sport/football/\#** subscription gets *both* publications. This is because when you subscribed you used a *multi-level wildcard* (\#) to indicate that you were interested in messages published to the **sport/football** topic or any of its children, so you will get both messages.

    On the other hand, the **\#/hursley** subscription gets only one.

    ![](./images/pots/mq/lab3/image48.png)

2.  Return to the Subscriptions view in the MQ Explorer. Select the **ALL_FOOTBALL_SUB** subscription, right-click and select **Status**.

    ![](./images/pots/mq/lab3/image49.png)

1.  The message count should have a count of the messages that were published on this topic. Click **Close** to close the status window.

    ![](./images/pots/mq/lab3/image50.png)

1.  Right-click the queue **ALL_FOOTBALL_Q** and select **Browse Messages**.

    ![](./images/pots/mq/lab3/image51.png)

2.  You should see three messages on the queue (or as many as you put in the amqspub test). Select one of the messages, right-click and choose **Properties**.

    ![](./images/pots/mq/lab3/image52.png)

3.  Click the **Named Properties** tab. From this display you can see the originating topic string.

    ![](./images/pots/mq/lab3/image53.png)

4.  Close the four or five open command windows as you will no longer need them.

    ***This concludes Lab 3.***

[Continue to Lab 4](mq_basic_pot_lab4.html)

[Return MQ Basic Menu](mq_basic_pot_overview.html)