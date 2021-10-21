---
title: Monitoring and Troubleshooting
toc: false
sidebar: labs_sidebar
folder: pots/mq-appliance
permalink: /mq_appl_pot_lab4.html
summary: Monitoring and Troubleshooting 
applies_to: [developer,administrator]
---



# Lab 4 - Monitoring and Troubleshooting

## Monitoring and troubleshooting

### Monitoring and reporting

For this lab, you should use the same CSIDE environment that you created
for Lab 1. The virtual appliance you will use for this lab will be
**MQAppl1** and also the **Windows 10 x64*** VM.

{% include important.html content="It is assumed that Lab 1 has been completed. You must either complete Lab 1 before attempting this lab, or see the *MQ Appliance PoT Configured - ready for HA* CSIDE template. Screen shots are from after Lab 2 completion. If Lab 2 has not been completed on the virtual appliance you are using, the results you see will differ from the examples in this lab guide." %}

You will be exploring some of the options available for monitoring the
MQ Appliance using a combination of command line and reporting in the MQ Console. You will also look at how you can implement the use
of the application activity trace for the MQ Appliance. Finally, you
will see some of the commands available for diagnosing and resolving
problems.

### Monitoring system resource usage

1.  Open the *MQAppl1* image, if it is not already running from previous
    labs.

	Before you investigate the MQ-specific resource monitoring, you will
start by monitoring the operation of the appliance itself. You can use
the *show* command to monitor different aspects of the appliance.

	You can use the show command to view information about how an aspect of
the appliance is configured or to monitor aspects of the appliance
operation. The status_provider argument specifies which information you
view. The show command is available at login, and in most configuration
modes.

	There is a large list of *status_provider* values available for the *show* command. The complete
list can be found in the IBM Knowledge Center for the MQ Appliance.

	You will investigate a few of these.

2. If you are at the *mqcli* command prompt, exit from it.

3. First, enter the **show** command. This will give a list of all of
    the available *status_provider* values.

4. Now enter **show version**. This will show the firmware and library version, similar to what is shown below.

	![](./images/pots/mq-appliance/lab4/image101.png)

	One of the other things that you can find out from the show command is
which users have been defined for the appliance.

5. Enter **show usernames**

6. You should see two users: the admin user, which was pre-configured,
    and the user that you set up in the first lab as the user authorized
    to reset passwords.

	![](./images/pots/mq-appliance/lab4/image10.png)

7. You can also use the show command to give a list of logged-on users.
    Enter **show users**.

	You can see below that the admin user is logged on via the serial
    port (which is you entering commands in to the console) and also
    logged on to the MQ Console (if you are not logged on, log on to the
    MQ Console [web admin] and rerun the command).

	![](./images/pots/mq-appliance/lab4/image11.png)

8. Next, get monitoring information about the MQ environment on the
    appliance.

    Go to the MQ command line interface.

	**`mqcli`**

9. Using **dspmq**, verify that the *QM1* from the first lab is running.
    If the queue manager is not running, start it.
    
    ![](./images/pots/mq-appliance/lab4/image11a.png)

	You will now use the status command to view information about
    resource allocation.
    
    {% include note.html content="About the status command: 
    
    
    You can use the status command to view the following information about the system resources on the appliance: 
    
    
    * the size and usage of the system memory of the system
     
    * the CPU usage of the system, the size and usage of the internal disk

     
    * the size and usage of the system volume

     
    * the number of FDCs and the disk space used

    
    * the disk space used by trace 
    " %}


10. Enter the command: **status**

	You will see a response similar to the following:

	![](./images/pots/mq-appliance/lab4/image12.png)
	
	 {% include note.html content="More about the status command: 
    
    
    You can also use the status command to view the following information about the system resources that are used by a queue manager: 
    
    
    * the queue manager name
     
    * the queue manager status

     
    * an estimate of the CPU usage of the queue manager

     
    * an estimate of the memory usage of the queue manager

    
    * the amount of the queue manager file system used by the queue manager

    
    
    For a high availability queue manager, additional information can be viewed: 
    
    
    * the file system size for the queue manager
     
    * the replication status of the queue managers

     
    * the preferred appliance for the queue manager

     
    * whether a partitioned situation has been detected

    
    * if detected, the amount of 'out-of-sync' data held 
 
    " %}


11. Enter the command for QM1:

	**`status QM1`**

12. You should see a response similar to the following:

	![](./images/pots/mq-appliance/lab4/image12a.png)
	
	You can use the amqsrua command to query metadata that is related to the
system resource usage of a queue manager.

	{% include note.html content="About amqsrua: 
    
    
    The amqsrua command reports metadata that is published by queue managers. This data can include information about the CPU, memory, and disk usage. You can also see data equivalent to the STATMQI PCF statistics data. The data is published every 10 seconds and is reported while the command runs. 
    
    
    * -n MaxPubs -- Specifies how many reports are returned before the command ends. The command publishes data every ten seconds, so if you enter a value of 50, the command returns 50 reports over 500 seconds.
	

    
    
    If you do not specify this parameter, the command runs until either an error occurs, or the queue manager shuts down.
	
	
	
        
    
    * -m QMgrName -- Specifies the name of the queue manager that you want to query. The queue manager must be running. If you do not specify a queue manager name, the default queue manager is used.   
	" %}

13. You are going to report the metadata for a minute, while putting
    some messages on the TEST.IN queue from the first lab, and see the
    results.

14. Enter the command as follows:

	**`amqsrua -n 30 -m QM1`**

	with the following responses:

	**STATMQI** for the Class selection and **PUT** for the Type selection.

	![](./images/pots/mq-appliance/lab4/image13.png)
	
15. Either go to the MQ Console or to the MQ Explorer (in the Windows
    image) and put some messages on the **TEST.IN** queue.

16. Depending on which method you use, and how many messages you put on
    the queue, you will see results similar to the following:

	![](./images/pots/mq-appliance/lab4/image14.png)
	
17. Enter **CTRL-C** to end the amqsrua command.

18. Now you will move on to the MQ Console and see some of the reports
    that you can generate.

    If you are not already logged on to the MQ Console on the Windows
    VM, do so now.

19. Return to the *MQ Console > Manage > QM1*. Click the elipsis on far right for the *TEST.IN* queue and select **Clear queue**. 

	![](./images/pots/mq-appliance/lab4/image15.png)

20. Click **Clear queue** to confirm deletion of the messages.

	![](./images/pots/mq-appliance/lab4/image16.png)

### Resource Monitoring

As you are monitoring persistent messages, you need to ensure that the default persistence on the queue is set to persistent, because the MQ Console will put the message using persistent as per the queue definition.

1. As we already have all of the connectivity for this test set up for
QM1, go to the MQ console and create a new local queue on **QM1** named **MONITOR**.

	![](./images/pots/mq-appliance/lab4/image104.png)
	
	![](./images/pots/mq-appliance/lab4/image105.png)
	
	![](./images/pots/mq-appliance/lab4/image106.png)
	
	![](./images/pots/mq-appliance/lab4/image107.png)	
12. Open the **Properties** of the **MONITOR** queue and change the
    default persistence to **Persistent** if it is not already set to
    that. Click *Save*.

	![](./images/pots/mq-appliance/lab4/image108.png)
	
	![](./images/pots/mq-appliance/lab4/image109.png)

1. As you saw in lab 1, some files and programs should have been provided
in the **perf** folder. These will again be used for this part of the
lab. The applications are part of SupportPac IH03. 

1. Go back to the command window you used earlier. If you closed it,
    open a **Command Prompt** window and change to the
    **C:\\utilities\\RFHutil\\** directory. 
    
    Open the **parmtst1.txt** parameters file in your favorite text
    editor, such as notepad.
    
    ![](./images/pots/mq-appliance/lab4/image110.png)

7. Make sure the **qmgr** parameter is:

    **USER.SVRCONN/TCP/10.0.0.1(1414)**

8. Change the **qname** parameter to **MONITOR**.

	![](./images/pots/mq-appliance/lab4/image36.png)
	
9. Change the **msgcount** to **1000** and change the **qmax** to **6000**.

	![](./images/pots/mq-appliance/lab4/image37.png)

10. Go to the bottom of the file and change the file in the
    **\[filelist\]** to **\Setup-Install\BigFile.xml**

	![](./images/pots/mq-appliance/lab4/image38.png)

11. Save the file (**Ctrl+S**).

12. Close the editor (notepad) session.

13. Go back to the command window.

14. Execute the **driver.cmd** file. 

	You will notice once the program starts that the queue depth
    settings are as follows:

	![](./images/pots/mq-appliance/lab4/image113.png)

16. Below is the output from the example we have just run (you may not
    get the same result -- but if not, you may wish to increase the
    number of messages).

	![](./images/pots/mq-appliance/lab4/image110.png)

17. Oops, we have run out of space for the queue manager!!! This is not
    likely to happen on a real appliance but you may recall that we have
    a very small queue manager file system size on our appliance). If
    the script did not end, hit **CTRL-C** to end the job.

20. Go back to the MQ Explorer. Here you see the thousand
    (probably 994) messages you have just put on the queue. There may be
    more if you did not clear out messages from previous tests.
    
    ![](./images/pots/mq-appliance/lab4/image112.png)

21. Clear the messages from the **MONITOR** queue on **QM1**.

### Troubleshooting

As the last section of this lab, you will review the logs that are
available for troubleshooting. The MQ Appliance has a set of logs
similar to traditional MQ. In this section, you will understand where
they are stored on the MQ Appliance and how to access them.

First, look at where the MQ error logs are stored.

1. On the *MQAppl1* appliance command line, exit from any command if
    necessary and then exit from mqcli. This will take you back to the
    **mqa\#** prompt.

2. Enter the **config** command to take you into configuration mode.

	**`config`**

	![](./images/pots/mq-appliance/lab4/image43.png)

	Directory structures on the appliance are accessible in the form of
    URIs. There is a dedicated URI, *mqerr*, for accessing IBM MQ logs.

3. Enter the following command (make sure you put the colon at the
    end).

	**`dir mqerr:`**

4. You will see the structure of the log directories.

	![](./images/pots/mq-appliance/lab4/image44.png)

5. Enter the following command to see the QM1 queue manager logs. These
    will be familiar to anyone familiar with MQ:

	**`dir mqerr:qmgrs/QM1`**

	![](./images/pots/mq-appliance/lab4/image45.png)
	
6. Enter **exit** to exit configuration mode.

7. Go to the MQ command line interface -- enter **mqcli**.

	You can list or view the system error logs, queue manager error logs,
and first failure data captures (FDCs) by using the dspmqerr command.

8. Enter:

	**`help dspmqerr`**

9. This will list the options available for viewing the logs.

	![](./images/pots/mq-appliance/lab4/image46.png)
	
10. Look at the AMQERR01.LOG for QM1. Enter:

	**`dspmqerr -m QM1 AMQERR01.LOG`**

	![](./images/pots/mq-appliance/lab4/image47.png)
	
	 {% include note.html content="About dspmqerr: 
    
    
    The command is based on the UNIX *less* command. The less command provides controls for navigating the contents of a file, and you can use these controls when you view system error logs.: 
    
    
    Try the following tips:

    
     
    * Use the arrow keys to scroll up and down the logs

     
    * Use the page, space, or return keys for simple scrolling

     
    * Enter *q* to exit at any time

    
    * Enter *h* to display full help while you view a log. The help lists further commands, for example, for searching for strings or jumping a set number of lines. 
    " %}
	
11. When done reviewing the log, enter **q**.

12. Enter this command to see the system log.

	**`dspmqerr -s`**

	![](./images/pots/mq-appliance/lab4/image48.png)

13. When done reviewing the log, enter **q**.

14. It is possible to copy the error logs from the MQ Appliance. This is
    done from the configuration mode (the config prompt) using the
    **copy** command or from the MQ Console (using the File Management
    option).

	We can also perform most of the file operations from the console.
    You did this in the Console lab -- if you did not get to it, take
    some time now to go back and complete the "tour" of the files.

## Extra Credit 

When we ran our previous test, we received an error for filling the
queue manager file system. In the console -- can you find it?

You can also run *strmqtrc* from the command line interface, and after,
the *endmqtrc* command. The trace files (AMQppppp.qq.TRC) can also be
copied from the MQ Appliance.


{% include note.html content="copy command: 
    
    
You will not be doing this in the lab; however, the command can be used as follows: 
    
    
<https://www.ibm.com/support/knowledgecenter/en/SS5K6E_9.1.0/com.ibm.mqa.doc/reference/filecmds/copy_flash.htm>

 " %}



Congratulations, this concludes the Monitoring and troubleshooting lab.