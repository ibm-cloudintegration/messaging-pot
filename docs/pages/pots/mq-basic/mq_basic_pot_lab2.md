---
title: Configuring the MQ JMS Provider
toc: false
sidebar: labs_sidebar
folder: pots/mq-basic
permalink: /mq_basic_pot_lab2.html
summary: Create MQ Objects - Send and Receive Messages
applies_to: [administrator,developer]
---

# Lab 2 - Configuring the IBM MQ JMS Provider 

The purpose of this lab is to demonstrate some of the typical steps you will go through when configuring IBM MQ as a JMS provider. Some of the tasks that you will perform include:

* Configure the Administered objects that a JMS program typically requires.
    Administered objects are used to externalize **Connection Factories** and
    **Destinations** from the program. This allows JMS Applications to be
    portable between Messaging Providers by shielding the applications from
    provider-specific details.

* Define a **Java Naming and Directory Interface** (**JNDI) directory.** In
    this lab you will perform the following:

    * Create a JNDI directory

    * Populate it with Connection Factory and Destination definitions

    * Use the MQ Explorer wizard to create corresponding MQ definitions

    * Run one of the Java™ Message Service (JMS) sample programs from the command line to use those definitions to connect to MQ as a JMS provider and produce JMS messages


[View an overview of using JMS with IBM MQ](https://ibm.box.com/s/w6c73p1tanrosfzyc8z75h8ui0cngxya)

## Create Administered Objects using MQ Explorer 

You may use the same demo environment that you used for Lab 1. If you are doing this lab consecutively following Lab 1, you can continue with that environment. If you are doing this lab independent of Lab 1, you may need to create a new demo environment. In that case click the link below.

[See the Environment Setup section](https://pages.github.ibm.com/cloudintegration/PoT-messaging/env_setup.html)    

 {% include note.html content="The following instructions assume that you are using the same demo environment from Lab 1. In this case you have created the **MQPOT** queue managers. If the MQPOT queue manager does not appear in the MQ Explorer, you must create it according to the instructions in Lab 1." %}
   

1. If the IBM MQ Explorer is not already running, you can launch it from the shortcut on the desktop.    
![](./images/pots/mq/lab2/image1.png)

1.  Start the IBM MQ Explorer by double-clicking the **MQ
    Explorer** shortcut.

2.  A directory has been created on your image called C:\\PoT-messaging\\MQ-POT\\JMS. This directory will hold the JNDI Namespace. When you create a connection to the namespace a file named .bindings will be created in this directory.

3.  Right-click **JMS Administered Objects** in the Navigator pane, and select **Add Initial Context…**.    
    ![](./images/pots/mq/lab2/image2.png)

4.  On the **Connection details** screen, click **File System**. Then click
    **Browse** to navigate to the directory called **C:\\PoT-messaging\\MQ-POT\\JMS**. Click **Next**.

    ![](./images/pots/mq/lab2/image3.png)

5.  On the **User preferences** screen, enter the Context nickname **Context1**.
    Note this can be a name of your choice but for this lab you will use
    **Context1**. This name will not be used elsewhere. Check both the **Connect immediately on finish** and **Automatically reconnect to context on startup** checkboxes, and then click **Finish**.

    ![](./images/pots/mq/lab2/image4.png)

6.  The newly created initial context is displayed in the list.    
    ![](./images/pots/mq/lab2/image5.png)

    This concludes this portion of Lab 2.

## Create a connection factory for IBM MQ

A connection factory is the mechanism used by JMS to manage connections between your JMS application and the JMS Provider. You will now define a
connection factory in the JNDI namespace.

1.  Expand the context name you just created. Right-click **Connection
    Factories.** Select **New Connection Factory**. 
    ![](./images/pots/mq/lab2/image6.png)

1.  Enter the name **CF1**. Note this can be any name of your choice but for
    this lab you will use CF1. This will be required when running the program so you might want to make note of it. Accept **IBM MQ** as the messaging provider and then click **Next \>**.    
    ![](./images/pots/mq/lab2/image7.png)

1.  On the next screen, accept the Type as **Connection Factory** and leave
    **Support XA transactions** unchecked. Click **Next \>**.   
    ![](./images/pots/mq/lab2/image8.png)

1. On the next screen, accept **Bindings** as the Transport given we will run
    the JMS Application on the same machine as the Queue Manager. Click **Next \>**.     
    ![](./images/pots/mq/lab2/image9.png)

1. On the next screen, leave **Create with attributes like an existing
    connection factory** unchecked. Click **Next \>**.

    ![](./images/pots/mq/lab2/image10.png)

1. This will open the **New Connection Factory** property sheet. Select
    **General** on the left hand menu **CF1** for *Name*. Use the pull-down to '8' for *Provider version*. This represents IBM MQ V8. Selecting this will enable JMS programs using this connection factory to utilize the new features of IBM MQ V8.   
    
    ![](./images/pots/mq/lab2/image11.png)
    
1. Select **Connection** on the left hand menu. Click the **Select** button for **Base queue manager**.
    
    ![](./images/pots/mq/lab2/image12.png)

1. Select the previously-defined Queue Manager **MQPOT** and then click **OK**.   
    ![](./images/pots/mq/lab2/image13.png)

1. Click **Finish** to create the connection factory.    
    ![](./images/pots/mq/lab2/image14.png)

1. Click **OK** to dismiss the confirmation box.    
    ![](./images/pots/mq/lab2/image15.png)

1. Observe that Connection Factory **CF1** now appears in the Context list.    
    ![](./images/pots/mq/lab2/image16.png)

    This concludes this portion of Lab 2.

## Create a Destination for the JMS Application to put a message onto

1. Right-click **Destinations** and click **New Destination…**.

    ![](./images/pots/mq/lab2/image17.png)

1. Enter the name **JMS1**. Note this can be any name of your choice but for this lab you will use **JMS1**. Leave the Type as **Queue** and ensure **Start wizard to create a matching MQ Queue** is checked. This will create
a corresponding IBM MQ Queue. The Queue that you create will be used
to verify that messages are successfully written using the sample program.
Click **Next \>**.    
    ![](./images/pots/mq/lab2/image18.png)

1. Accept the defaults on this next screen and click **Next \>**.    
    ![](./images/pots/mq/lab2/image19.png)

1. On the change properties screen, you can set some MQ-specific properties of the destination. Select **MQPOT** as the Queue Manager and define a Queue with the same name as the JMS Destination. Note the Queue name can be any name of your choice but for this lab you will use **JMS1**. This will be created as part of the upcoming wizard. Click **Finish** to launch the MQ configuration wizard.    
    ![](./images/pots/mq/lab2/image20.png)

1. Click **OK** to dismiss the confirmation screen.    
    ![](./images/pots/mq/lab2/image21.png)

1. The **Create an MQ Queue** wizard will start automatically. Click **Next
    \>**.     
    ![](./images/pots/mq/lab2/image22.png)

1. Accept the default **Local Queue** for Type and then click **Next \>**.   
    ![](./images/pots/mq/lab2/image23.png)

1. Click **Finish** to create the MQ queue.    
    ![](./images/pots/mq/lab2/image24.png)

1. Click **OK** to close the confirmation prompt.   
    ![](./images/pots/mq/lab2/image25.png)

1. Click the **Queues** folder under MQPOT in the MQ Explorer Navigator pane.
    Look at the Content pane to confirm that an MQ local queue called **JMS1** has been created.    
    ![](./images/pots/mq/lab2/image26.png)

This concludes this portion of Lab 2.

## Writing a JMS message using a Java sample program

Some very good code samples ship with IBM MQ. You will use one of them to create a JMS message on an MQ JMS queue.

1.  Open a **command prompt**. A shortcut for this is on the desktop.    
    ![](./images/pots/mq/lab2/image27.png) 
    
1.  Initialize environment with the following command:

     ```
     setmqenv -s
     ```

2.  Change the directory to C:\\Program Files\\IBM\\MQ\\Tools\\jms\\samples. Enter the following commands to add **Java** to the *Path* and the required *JMS jar files* to your *Classpath*. 

    ```
    set PATH=%PATH%C:\Program Files\Java\jre1.8.0_jre
    ```
    
    ```
    set CLASSPATH=%CLASSPATH%C:\Program Files\IBM\MQ\java\lib64;C:\Program Files\IBM\MQ\java\lib\com.ibm.mqjms.jar;C:\Program Files\IBM\MQ\java\lib\com.ibm.mq.jar;.;
    ```
    
1. Now run the sample application:

    ```
    java JmsJndiProducer -i file:/c:/Pot-messaging/MQ-POT/JMS -c CF1 -d JMS1
    ```

    This will run the Java JMS sample program JmsJndiProducer. The **“-i”**
    argument points the program to the location of your JNDI directory. The
    **“-c CF1”** identifies the connection factory for your test queue manager, and the **“-d JMS1”** identifies the JMS destination queue.   
    ![](./images/pots/mq/lab2/image28.png)

    Having run this program, you should now have a message in the **JMS1** queue. To see whether you do, you can look for the message that was produced by this program using the IBM MQ Explorer. Switch back to the MQ Explorer window.

1.  In the MQ Explorer Navigator pane, locate the folder called **JMS
    Administered Objects**. Click the **Destinations** folder beneath it, and in the Content pane you should see your JMS Destination **JMS1**. Double-click the **JMS1** destination to find the MQ Queue that is associated with it.    
    ![](./images/pots/mq/lab2/image29.png)

1.  Recall that MQ queue **JMS1** is associated with the JMS destination
    **JMS1**. Click **Cancel.**    
    ![](./images/pots/mq/lab2/image30.png)

1.  Right-click the **JMS1** MQ queue object and then select **Browse
    Messages…**.   
    ![](./images/pots/mq/lab2/image31.png)

1.  The **message data** column in the display should match the message text
    that you wrote with the JmsJndiProducer program.      
    ![](./images/pots/mq/lab2/image32.png)

1.  Click **Close** to close the **Message browser**.    
    ![](./images/pots/mq/lab2/image33.png)

    This concludes Lab 2.

[Continue to Lab 3](mq_basic_pot_lab3.html)

[Return MQ Basic Menu](mq_basic_pot_overview.html)
