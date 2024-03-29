---
title: File Transfer Patterns
toc: false
sidebar: labs_sidebar
folder: pots/mq-mft
permalink: /mq_mft_pot_lab3.html
summary: Demonstrate 13 common patterns using MQ MFT
applies_to: [developer,administrator]
---

# File Transfer Patterns 

IBM MQ Managed File Transfer (MFT) uses the Apache Ant scripting language for transfer tasks which require more complex processing such as multiple transfers, conditional processing, and building a process flow of interdependent transfers. This lab will showcase the power of combining Ant with MFT. Included are several typical patterns of complex transfer processing. These patterns have been compiled from actual customer implementations. The patterns were described in the presentation preceding this lab. The patterns available for exercise are:

* Asynchronous transfer with await outcome

* Multi step transfer job with program call

* Transfer single file to multiple end points

* Pull files from two spokes of a hub and concatenate into a single file

* Use PGP with MFT

* Dynamically set job name and MFT metadata based on file contents

* Use a custom Ant task to validate the contents of a file

* Transfer single file to multiple end points using RegEx to dynamically build the distribution list

* Split a “stack” file using JScript and then programmatically invoke Ant to transfer each generated file

* Invoke a remote Ant job to have it pull a file

* Unzip two files each containing multiple files, concatenate all extracted files into a single file and send to remote agent

* Transform CSV files to XML using WMB

## IBM MQ Managed File Transfer Ant Extensions

IBM MQ Managed File Transfer includes a number of extensions for Apache Ant which enables you to carry out certain tasks. MQ File MFT provides tasks that you can use to integrate file transfer function into the Apache Ant tool.

MQ MFT provides a number of Ant tasks that you can use to access file transfer capabilities.

* [fte:awaitoutcome](http://publib.boulder.ibm.com/infocenter/wmqfte/v7r0/topic/com.ibm.wmqfte.doc/await_outcome.htm)

* [fte:call](http://publib.boulder.ibm.com/infocenter/wmqfte/v7r0/topic/com.ibm.wmqfte.doc/call.htm)

* [fte:cancel](http://publib.boulder.ibm.com/infocenter/wmqfte/v7r0/topic/com.ibm.wmqfte.doc/cancel.htm)

* [fte:filecopy](http://publib.boulder.ibm.com/infocenter/wmqfte/v7r0/topic/com.ibm.wmqfte.doc/file_copy.htm)

* [fte:filemove](http://publib.boulder.ibm.com/infocenter/wmqfte/v7r0/topic/com.ibm.wmqfte.doc/file_move.htm)

* [fte:ignoreoutcome](http://publib.boulder.ibm.com/infocenter/wmqfte/v7r0/topic/com.ibm.wmqfte.doc/ignore_outcome.htm)

* [fte:ping](http://publib.boulder.ibm.com/infocenter/wmqfte/v7r0/topic/com.ibm.wmqfte.doc/ping.htm)

* [fte:uuid](http://publib.boulder.ibm.com/infocenter/wmqfte/v7r0/topic/com.ibm.wmqfte.doc/uuid.htm)

The following nested parameters describe nested sets of elements, which are common across several of the supplied Ant tasks:

* [fte:filespec](http://publib.boulder.ibm.com/infocenter/wmqfte/v7r0/topic/com.ibm.wmqfte.doc/filespec.htm)

* [fte:metadata](http://publib.boulder.ibm.com/infocenter/wmqfte/v7r0/topic/com.ibm.wmqfte.doc/ant_metadata.htm)

You can use the fteAnt command to run Ant tasks in a MQ MFT that you have already configured. Using Ant scripts with MQ MFT allows you to coordinate complex file transfer operations from an interpreted scripting language.

The fteAnt command runs Ant scripts in an environment that has IBM MQ Managed File Transfer Ant tasks available. Unlike the standard ant command, fteAnt requires that you define a script file. Ant scripts (or build files) are XML documents defining one or more targets. These targets contain task elements to run.

Examples of Ant scripts that use IBM MQ Managed File Transfer tasks are provided with your product installation in the directory *install\_dir*/samples/fteAnt.

A namespace is used to differentiate the file transfer Ant tasks from other Ant tasks that might share the same name. You define the namespace in the project tag of your Ant script. 
   
    <?xml version="1.0" encoding="UTF-8"?> 
    <project xmlns:fte="antlib:com.ibm.wmqfte.ant.taskdefs"
     default="do_ping"> 
       <target name="do_ping"> 
          <fte:ping cmdqm="qm@localhost@2414@SYSTEM.DEF.SVRCONN" 
          agent="agent1@qm1" rcproperty="ping.rc" timeout="15"/> 
       </target>
    </project>


The attribute *xmlns:fte="antlib:com.ibm.wmqfte.ant.taskdefs"* tells Ant to look for the definitions of tasks prefixed by fte in the library *com.ibm.wmqfte.ant.taskdefs*.

You do not need to use *fte* as your namespace prefix; you can use any value. The namespace prefix *fte* is used in all examples and sample Ant scripts.

To run Ant scripts that contain the file transfer Ant tasks use the fteAnt command. For example:

 fteAnt -file *ant\_script\_location*/*ant\_script\_name*

The file transfer Ant tasks return the same return codes as the IBM MQ Managed File Transfer commands.

### Create new agents for Lab 3

For Ant scripts, you will use a client agent on Win10 to connect to local agents on win16. First you will need to create additional agents for the scripts to run. We have provided Windows commnand files to create the additional agents.

1. On the Teacher machine (win16) open the Windows explorer and navigate to *C:\PoT-messaging\IBM\MFT*.

	![](./images/pots/mq-mft/lab3/image50.png)
	
1. Double-click *Create-Agents*. This script will define local MFT agents **MFT4AGT2** and **MFT4AGT3**. Follow the instructions as the script displays each step and pauses until you continue. 

	![](./images/pots/mq-mft/lab3/image51.png)
	
	The script will create the agents, execute *runmqsc* to define the agents' queues and start the agents.
	
1. In a command prompt enter the command:

	```
	fteListAgents
	```
	
	![](./images/pots/mq-mft/lab3/image52.png)
	
	You can see the agents you created in the previous labs and the new agents created by the script.
	
1. In MQ Explorer, expand *Managed File Transfer*, connect to the coordination queue manager *MFT5* then click *Agents*.

	![](./images/pots/mq-mft/lab3/image53.png)
	
1. Switch to the Student machine (win10) to create the client agent. Open the Windows Explorer and navigate to *C:\PoT-messaging\IBM\MFT*.

	![](./images/pots/mq-mft/lab3/image54.png)
	
1. Double-click *ClientAgent* to create the agent and send the *MFT4CLT1_create.mqsc* file to win16 using MQ MFT Agent *CSM1AGT1* on win10 to send and MQ MFT Agent *MFT4AGT1* on win16 to receive. 

	![](./images/pots/mq-mft/lab3/image55.png)
	
1. In MQ Explorer, click *Managed File Transfer > MFT5 > Transfer Log*. You will see the transfer highlighted in green with *Completion State* **Successful**. 

	![](./images/pots/mq-mft/lab3/image56.png)

1. Expand the the transfer to see the file transferred. At the bottom of screen you will see the *Current Transfer Progress*. This display more detailed information such as number of files, size of files, and time stamp. If the transfer was still in progress you would see the progress bar move as data is received.

	![](./images/pots/mq-mft/lab3/image57.png)

1. Switch to the Teacher machinve (win16). In a command window, navigate to *C:\ProgramData\IBM\MQ\mqft\config\MFT5\agents\MFT4CLT1*. 

	```
	cd \ProgramData\IBM\MQ\mqft\config\MFT5\agents\MFT4CLT1
	```

	![](./images/pots/mq-mft/lab3/image58.png)
	
1. Enter the *dir* command to show the directory members. You will see the transferred file *MFT4CLT1_create.mqsc*. Execute *runmqsc* on MFT4 to create the agent queues for agent MFT4CLT1. MFT4 is the agent queue manager.

	![](./images/pots/mq-mft/lab3/image59.png)
	
1. As you did previously, open *defineqs.txt* in Notepad++ to verify the queues were created successfully.

	![](./images/pots/mq-mft/lab3/image63.png)

1. Return to MFT Client machine (win10). You can now start the agent. In the Windows Explorer where you have *C:\PoT-messaging\IBM\MFT* open, double click *AgentStart*.

	![](./images/pots/mq-mft/lab3/image60.png)
	
1. Enter the following command to display the agents:

	```
	fteListAgents
	```
	
	![](./images/pots/mq-mft/lab3/image61.png)
		
1. Display the agents in MQ Explorer. 

	![](./images/pots/mq-mft/lab3/image62.png)		
	Both show that agent MFT4CLT1 is ready to go. You will see the same display on the MFT Server machine (win16).
	
Let's review what happened. 
	
1. In Windows Explorer, navigate to C:\PoT-messaging\IBM\MFT. Right-click *ClientAgent.cmd* and select *Edit with Notepad++*. 

	![](./images/pots/mq-mft/lab3/image64.png)
	
1. In the same manner open *CreateClientAgent.cmd* and *SendClientQs.xml* in Notepad++ so you can review all three files.

1. In Notepad++ highlight *ClientAgent.cmd*. This command file sets the variables to be used and calls *CreateCLientAgent.cmd* and then calls *fteAnt* with the -f parameter which indicates the Ant script to use. Ant scripts are XML files.

	![](./images/pots/mq-mft/lab3/image65.png)

1. Highlight *CreateClientAgent.cmd*. This is the MQ MFT command to create an agent. You used this command in Lab 1. The parameters are substituted from the calling program (ClientAgent.cmd). However you will notice the additional parameters agentQMgrHost, agentQMgrChannel, and agentQMgrPort. These are required to connect to the agent queue parameter MFT4 which is running on another host.

	
	
	
## IBM Integration Bus Toolkit Resource Perspective 

This POT uses the IBM App Connect Toolkit to run the transfer pattern Ant scripts. The Integration toolkit was chosen because it is installed on the PoT VMware image. We will use the Eclipse Resource Perspective in the toolkit to analyze and run the scripts. Eclipse is a very effective development platform because it provides syntax assistance for Ant and XML. Eclipse is not required. In fact all you need is a text editor to create Ant scripts. But Eclipse makes it much easier for developing and test Ant scripts.

 {% include note.html content="*Using the IBM Integration Toolkit is not required*. 
 You can freely download and Eclipse platform from [www.eclipse.org](http://www.eclipse.org)." %}

1.  On the Student image, start IBM Integration Toolkit by double-clicking the icon on the desktop.

    ![](./images/pots/mq-mft/lab3/image1.png)

    Enter *C:\\PoT-messaging\\MQ-POT\\MFT\\workspace* in the Workspace field. If the directory or file does not exist it will be created. Click *OK*.

    ![](./images/pots/mq-mft/lab3/image2.png)

2.  If the *Welcome* window opens, click the arrow to close the *Welcome* window and *Go to the Message Broker Toolkit*.

    ![](./images/pots/mq-mft/lab3/image3.png)

    We will use the *Resource* perspective for the *Transfer Patterns*. If the toolkit opens in the Resource perspective and shows the project **MFTant** in the Project Explorer as shown here, then skip to *Step 5*.

    ![](./images/pots/mq-mft/lab3/image4.png)

3.  The toolkit should open in the *Resource* perspective. If not, click the *Window* pull-down menu and select *Open Perspective** \> *Other**.

    ![](./images/pots/mq-mft/lab3/image5.png)

4.  Select *Resource*.

    ![](./images/pots/mq-mft/lab3/image6.png)

5.  When the *Resource Perspective* opens, you will see the *MFTant* project in the *Project Explorer*.

    Expand *MFTant*.

    ![](./images/pots/mq-mft/lab3/image7.png)

6.  Expand *Demos* \> *DemoUseCases*. Scroll down until you reach the **UseCase**…. Files.

    ![](./images/pots/mq-mft/lab3/image8.png)	
## Pattern 1 - Asynchronus Transfer with Await Outcome 

This transfer pattern demonstrates how an Ant Transfer Template can be used to send a file in *asynchronous* mode and then *await* the outcome. The fte:awaitoutcome task is one of the MFT extensions and is used to wait for the outcome of a MFT transfer.

The pattern demonstrates how one can setup an fteAnt template to be able to transfer a file to one of three types of endpoints. The following end points can each be tested: MFT agent **FTE2008R2**, FileZilla FTP Server **FTE2008R2** via the **FTP\_BRIDGE** agent, and Connect:Direct node **FTE2008R2** via the **CD\_BRIDGE**.

  ![](./images/pots/mq-mft/lab3/image9.png)
 
 *Figure 3.3.1 – Pattern 1 – Asynchronous Transfer with Await Outcome*

This pattern uses the protocol bridge agent as well as the native MFT agent. The protocol bridge enables your IBM MQ Managed File Transfer network to access files stored on a file server outside your IBM MQ Managed File Transfer network. This file server can use the FTP, FTPS, or SFTP network protocols. Each file server needs at least one dedicated IBM MQ Managed File Transfer agent. The dedicated agent is known as the protocol bridge agent. See the Figure 3.3.2 below.

The protocol bridge is available as part of the Server component of IBM MQ Managed File Transfer. You can have multiple dedicated agents on a single system running IBM MQ Managed File Transfer Server that connect to different file servers.

 ![](./images/pots/mq-mft/lab3/image10.png)
 
 *Figure 3.3.2 – MFT Protocol Bridge Architecture*

### Reference Material for Pattern 1 

The following commands are examples which show how one invokes MFT Ant and overrides the default properties of the template. Pay particular attention to the TNODE parameter which indicates the destination agent involved in the transfer.

![](./images/pots/mq-mft/lab3/image11.png)

*Figure 3.3.3 – POT Command file for MFT Pattern 1 Using ConnectDirect Bridge Agent*

![](./images/pots/mq-mft/lab3/image12.png)

*Figure 3.3.4 – POT Command file for MFT Pattern 1 Using FTE Agent*

![](./images/pots/mq-mft/lab3/image13.png)

*Figure 3.3.5 – POT Command file for MFT Pattern 1 Using FTP Bridge Agent*

![](./images/pots/mq-mft/lab3/image14.png)

*Figure 3.3.6 – Ant Script (template) for running MFT Pattern 1, UseCase1.xml*

### Review the Ant script UseCase1.xml 

1.  Double click **UseCase1.xml** to open it in the **Editor** panel.

2.  Double-click the **UseCase1.xml** tab to maximize the edit pane for better viewing.

    ![](./images/pots/mq-mft/lab3/image15.png)

3.  You are looking at an Ant script – XML file. The script can also be referred to as the Ant template. You may also refer to Figure 3.3.6. There are a few things we need to observe in order to understand how Ant functions.

* **XML version** is declared on first line.

* The Ant script is contained within a **\<project\>** tag.

* **\<description\>** tags describe the function the script will perform.

* Comments are embedded between **\<!--** and **--\>** tags and show up in a different color than the markup and values.

* **Ant** tasks are coded within a **target** tag. There are four targets in this script named:

    * init
    
    * step1
    
    * step2
    
    * job

* **fte:xxxxxxx** are Ant extensions for **WMQ MFT** and provide translation from Ant to MFT functions.

* Values can be over-ridden at run time.

 For anyone who is familiar with **JCL**, you may discover a similarity between targets and JCL **STEP**s, **fte:xxxxxx** parameters and **DD** parms. Targets can be conditioned on previous targets just as STEPs in JCL can be (look for the “**depends”** parameter on the targets). The *init* target is very comparable to a COBOL Working-Storage Section.

 Still referring to the Ant script or Figure 3.3.6 above, notice the steps (targets or tasks) of the script and the fte: commands that are being issued by the script. You can see that the script executes multiple steps to complete the transfer.

* Target *init* is our housekeeping step that assigns values to variables (properties). The default values are defined here and can be over-ridden at runtime.

* *step1* is dependent on *init* and executes a **fte:copy** of the file from the agent STUDENT1 to the WMQ2008R2DC agent. A **fte:copy** does not delete the source file. Meta-data is defined with the **fte:metadata** stanza. Meta-data are name-value pairs of data about the transfer which is carried with the transfer. Each pair of meta-data is defined by the **fte:entry** command and can also be over-ridden at runtime. **fte:filespec** indicates the source file(s), destination file(s), and whether destination files should be overwritten.

* *step2* is dependent on successful completion of the *step1* and awaits the outcome of the transfer with the ID of the transfer (assigned in step1). This is a substitution variable that gets resolved at runtime. It has a timeout value set at 10 mins (600 seconds).

1.  Double-click the **UseCase1.xml** tab to return to the normal view.

    ![](./images/pots/mq-mft/lab3/image16.png)

### Run Script from MFT Eclipse workbench 

1.  In the Project Explorer, right click the file **UseCase1.xml**.

2.  Select Run As \> Ant Build.

    ![](./images/pots/mq-mft/lab3/image17.png)

    {% include note.html content="You may also right-click inside the edit panel of UseCase1.xml to get the menu where you can select Run As." %}

1.  Click the **Console** tab to view the results of the transfer. You will see the the script starting. An Ant script is referred to as a Buildfile in the Ant runtime.

    ![](./images/pots/mq-mft/lab3/image18.png)

2.  Double click the Console tab to view the log in full-screen mode. This will give you a better view and you will be able to see the entire log.

    ![](./images/pots/mq-mft/lab3/image19.png)

    Notice that the file was transferred to the FTP Server via the **FTP\_BRIDGE** agent. You can see that the BUILD (Ant script) was successful and took six seconds to run.

3.  Double-click the Console tab again to return it to normal size.

### Review results on MFT Server 

The agents report status messages about the transfer to the coordination queue manager – MB8QMGR on the MFT Server. The coordination queue manager publishes these messages on behalf of the agents. The MFT plugin for MQ Explorer subscribes to the publications and summarizes them in the MFT Transfer Log and the MFT – Current Transfer Progress panes.

1.  Switch to the MFT Server and open **MQ Explorer**.

2.  In the **MQ Explorer**, click**Transfer Log** under the Managed File Transfer folder.

    ![](./images/pots/mq-mft/lab3/image20.png)

    You will see a successful transfer from agent STUDENT1 to the FTP\_BRIDGE protocol agent.

3.  Expand the transfer by clickingthe plus sign next to the Source agent STUDENT1.

    ![](./images/pots/mq-mft/lab3/image21.png)

4.  Open **Windows Explorer** on the MFT Server and navigate to **C:\\tmp\\From\_STUDENT1**.

5.  You will find the file Big1.zip which was transferred from STUDENT1.

    ![](./images/pots/mq-mft/lab3/image22.png)

### Run Script as batch file - runtime substitution 

The Eclipse Workbench is not required to run Ant scripts. It is excellent for developing and testing Ant scripts. But in most cases, Ant scripts will be run from a command prompt submitted from scheduling systems or other programs. The same Ant scripts can be run by a command which overrides variables at runtime.

In this exercise you will rerun UseCase1.xml with different commands overriding the agents and protocols used. UseCase1.xml was coded to use the FTP protocol and transfers files from agent STUDENT1 to FTP\_BRIDGE on the MFT server. Those are the default settings.

In the **POTCommands** subdirectory, there are commands to send the same file to the MFT Server using the ConnectDirect protocol bridge agent and also to an MFT agent on MFT Server.

1.  Return to the Student machine.

2.  Open POT Commands by double-clicking the icon on the desktop.

    ![](./images/pots/mq-mft/lab3/image23.png)

1.  Right-click the **UseCase1FTE** file and select **Edit in Notepad++**. Notepad++ is a free editor comparable to Ultraedit and is an excellent tool for viewing and editing XML files, Windows command and batch files, etc.

    ![](./images/pots/mq-mft/lab3/image24.png)

2.  Notepad++ opens the file in edit mode and you can see the commands that will be issued when the command is run.

    ![](./images/pots/mq-mft/lab3/image25.png)

3.  Examine the commands. There are only three commands:

* cd change directory

* call

* pause

    The rest of the lines are parameters for **fteAnt**. The “cd” just changes the current directory to the directory where the transfer patterns are located – C:\\IBM\\fteAnt\\Demos\\DemoUseCases. The call command invokes the Java program “fteAnt”.

1.  Arrange Notepad++ window and the WMB Toolkit window so you can see both like shown here.

    ![](./images/pots/mq-mft/lab3/image26.png)

    Here you can see the **UseCase1FTE.cmd** is invoking **fteAnt** and telling it to run **fteAnt** Java program with the **UseCase1.xml** script. The runtime overrides are indicated in blue. So even though the script has default values shown in the toolkit, the command file is overriding some of the values. “-D” is a Java convention for indicating parameters. So the property names in the script which are being overridden are jobname, srcfile, dstfile, SNODE, and TNODE. SNODE and TNODE are actually the agent names which will perform the transfer. The MFT convention is “agent name @ agent queue manager”.

    Take note, that for batch processing, it is the command file that should be started. Scheduling batch jobs is an alternative to MFT monitors or directory monitors. It depends on your processing requirements.

2.  Minimize the Windows Explorer, Notepad++, and WMB Toolkit.

3.  This time you will run UseCase1 as a batch command to run the Ant script, overriding the FTP\_BRIDGE agent with a MFT agent. In the Windows Explorer POTCommands panel, double-click UseCase1FTE.

    ![](./images/pots/mq-mft/lab3/image27.png)

4.  A Windows command window will appear where you will find the console messages. When running the Ant script from a batch command, the messages will appear in the command window instead of the WMB Toolkit console.

    Review the messages. Notice the commands being issued and the messages which are logged. You will see a message for copying the file from agent STUDENT1 to FTE2008R2. This is what we should expect since we are using the MFT protocol this time instead of the FTP protocol. The copy operation is assigned a unique **transfer ID**. Remember the three Ant tasks we saw in the script – **init**, **step1**, and **step2**. Step2 waits for the outcome of the transfer operation to be reported and checks that the operation was successful.

    ![](./images/pots/mq-mft/lab3/image28.png)

5.  Switch to the MFT Server and the Transfer Log. You will see a new successful transfer from STUDENT1 to FTE2008R for the same file Big1.zip.

    ![](./images/pots/mq-mft/lab3/image29.png)

### Run Script from command line

1.  You have now used the Eclipse Workbench and a batch command file to run the Ant scripts. Another way to run the script is to manually issue the command from the command line prompt. In this next section you will run the UseCase1.xml script from the command line over-riding the default values.

    In the previous exercises you ran UseCase1.xml with the FTP and MFT protocols. This time you will run the script using the Connect:Direct protocol.

1.  Return to the Student machine.

2.  Open a **Windows Command Prompt** from the taskbar by double-clickingthe icon.

    ![](./images/pots/mq-mft/lab3/image30.png)

3.  Change the directory to C:\\IBM\\fteAnt\\demos\\DemoUseCases by entering the command

    *cd \\IBM\\fteAnt\\demos\\DemoUseCases *

    ![](./images/pots/mq-mft/lab3/image31.png)

4.  Return to the Windows Explorer window for **PoTCommands**.

5.  Right click **UseCase1CD** and select Edit with Notepad++.

    ![](./images/pots/mq-mft/lab3/image32.png)

1.  When the editor opens, select lines 2 -6 and press **Ctrl+A** to highlight those lines and **Ctrl+C** to copy them.

    ![](./images/pots/mq-mft/lab3/image33.png)

2.  Return to the Command Prompt, right-clickthe empty line and select **Paste** and press Enter.

    ![](./images/pots/mq-mft/lab3/image34.png)

3.  Review the messages. Notice the commands being issued and the messages which are logged. You will see a message for copying the file from agent STUDENT1 to CD\_BRIDGE@MB8QMGR. The destination agent in this transfer is the Connect\_Direct protocol agent. The copy operation is assigned a unique **transfer ID**. Remember the three Ant tasks we saw in the script – **init**, **step1**, and **step2**. Step2 waits for the outcome of the transfer operation to be reported and checks that the operation was successful just as it did with the other protocols.

    ![](./images/pots/mq-mft/lab3/image35.png)

4.  Switch to the MFT Server and the Transfer Log. You will see a new successful transfer from STUDENT1 to WMQ2008R2DC(via CD\_BRIDGE) for the same file Big1.zip.

    ![](./images/pots/mq-mft/lab3/image36.png)

### Review results using IBM Sterling Control Center 

1.  Follow the steps in **Appendix A** to review your results in IBM Sterling Control Center.

### Summary 

In this section of the lab you have exercised three ways to submit MFT transfers – Eclipse workbench, batch file, and the command line. There are really four ways to submit transfers. In addition to the previous methods, you can also submit a transfer by putting a correctly formatted XML command message on an agent’s command queue or an application may put the command message on the command queue. We will not use this method in this lab.

For the transfers you performed, you transferred files using three protocols – MFT via the MFT agent FTE2008R2, ftp via the FTP\_BRIDGE agent and ConnectDirect via the CD\_BRIDGE.

{% include important.html content="For patterns 2 – 13, you are to choose one or more patterns which interest you as time allows.
The Ant scripts will be explained, but the detailed steps will not be repeated. Perform the exercises as were done in 3.3.1 – 3.3.7 substituting the appropriate UseCase / Pattern files.
Use any one of the three previous methods to submit the transfer requests.                                                                                                                 " %}


## Pattern 2 - Multi step transfer job with program call 

This pattern demonstrates the use of another MFT Ant extension. You can use the **fte:call** task to remotely call scripts and programs. This task allows you to send a **fte:call** request to an agent. The agent processes this request by running a script or program and returning the outcome. The commands to call must be accessible to the agent; that is the command’s path must be available to the agent and the agent must be permitted to execute the command.

In this exercise, you will see how a single Ant transfer template can be used to perform multiple steps as follows:

* Transfer multiple files from MFT Agent STUDENT1 to MFT Agent TEACHER

* Call the “NETSTAT” program at the TEACHER Agent

* Transfer multiple files from MFT Agent TEACHER to MFT Agent STUDENT1

    ![](./images/pots/mq-mft/lab3/image37.png)

    *Figure 3.4.1 – Pattern 2 – Multi Step Transfer Job with Program Call*

### Reference Material for Pattern 2 

 ![](./images/pots/mq-mft/lab3/image38.png)

 *Figure 3.4.2 – POT Command file for MFT Pattern 2*

 ![](./images/pots/mq-mft/lab3/image39.png)
 ![](./images/pots/mq-mft/lab3/image40.png)
 ![](./images/pots/mq-mft/lab3/image41.png)
 
 *Figure 3.4.3 – Ant Script for running Pattern 2, UseCase2.xml*

### Review the Ant script UseCase2.xml

Notice the steps (targets or tasks) of the script and the fte: commands that are being issued by the script.

You can see that the script executed multiple steps to complete the transfer.

* Target *init* is our housekeeping step that assigns values to properties (variables). The default values are defined here and can be over-ridden at runtime.

* *step1* is dependent on *init* and executes a **fte:copy** of the file(s) from the agent STUDENT1 to the WMQ2008R2DC agent. A **fte:copy** does not delete the source file. Meta-data is defined with the **fte:metadata** stanza. Meta-data are name-value pairs of data about the transfer which is carried with the transfer. Each pair of meta-data is defined by the **fte:entry** command and can also be over-ridden at runtime. **fte:filespec** indicates the source file(s), destination file(s), and whether destination files should be overwritten.

* *check1* is dependent on *step1* and checks return codes to either fail the script or let it continue.

* *step2* is dependent on successful completion of the *step1* and issues **fte:call** to agent WMQ2008R2DC to execute the “Netstat” command.

* *step3* is then run depending on the successful completion of *step2. step3* issues **fte:move** to move the file(s) from WMQ200R2DC to the agent STUDENT1 on the student’s VMware image. A **fte:move** deletes the file from the source machine. In the case of *step3*, WMQ2008R2DC agent is the source. Again, take note of the meta-data and filespec parameters.

### Run Pattern Script 

1.  Run the Ant script with your preferred method.

2.  Monitor runtime console – Console in the toolkit or command prompt window console.

### Review results on MFT Server

1.  Analyze the Transfer Log.

2.  Find proof that the file(s) were transferred.

### Review results using IBM Sterling Control Center 

1.  Follow the steps in **Appendix A** to review your results in IBM Sterling Control Center.

## Pattern 3 - Transfer single file to multiple end points 

This pattern demonstrates how to use an Ant transfer template to send one file to one or more endpoints. The script uses a pipe delimited file as a distribution list to send a single file to multiple MFT agents. Referring to Figure 3.5.1:

* The distribution list is in a format which uses a pipe delimited record for each destination end point / destination directory combination

* Transfers are assigned the same Job Name & Job Number metadata to allow all transactions to be correlated to simplify transfer tracking

    ![](./images/pots/mq-mft/lab3/image42.png)
    
    *Figure 3.5.1 – Pattern 3 - Transfer single file to mulitple endpoints*

### Reference Material for Pattern 3    
 ![](./images/pots/mq-mft/lab3/image43.png)
 
 *Figure 3.5.2 – POT Command file for running MFT Pattern 3*

 ![](./images/pots/mq-mft/lab3/image44.png)
 
 ![](./images/pots/mq-mft/lab3/image45.png)
 
 *Figure 3.5.3 – Ant Script for running MFT Pattern 3, UseCase3.xml*

 ![](./images/pots/mq-mft/lab3/image46.png)
 
 *Figure 3.5.4 – List1.distributionList*

 ![](./images/pots/mq-mft/lab3/image47.png)

 *Figure 3.5.5 – JavaScript distList1.js*
 
 ![](./images/pots/mq-mft/lab3/image48.png)
 
 *Figure 3.4.6 – JavaScript distList2.js*

### Review the Ant script UseCase3.xml 

#### Calling JavaScript from Ant Script 

In summary, this pattern needs to transfer a file to multiple endpoints. The file to be transferred will be substituted at run time into the reference variable `$`{srcfile}.
    
The transfer will use a distribution list built dynamically from a text file.

Referring to Figure 3.5.2, this is the command file, Windows \*.cmd or \*.bat file, to run the Ant script. It simply contains two commands:

* Change directory to location of the UseCaseN.xml files for the patterns

* Execute the call command to execute fteAnt with the –f parameter to specify the xml file to run

 The rest of the lines are overrides for properties set in the xml file. The java convention for overriding properties is “-Dproperty=xxxxxxx”. The example below shows the property **srcfile** being overridden and set to C:\\Demo\\disk1\\File4.txt.    
 
 *-Dsrcfile=c:\\Demo\\disk1\\File4.txt*
 
 When the script runs, this is the file to be sent to all endpoints in the distribution list. Take note of the other properties.

 `$`{listName} resolves to text file **C:\\IBM\\fteAnt\\Demos\\DemoUseCases\\List1.distributionList**. *Figure 3.5.4* shows the contents of the file. The records in the distribution list define the target agent and destination directory for each end point. The agent is represented in the MFT format *agent@queuemanager.* A pipe delimiter is used to separate the agent and the destination directory.

 Figure 3.5.3 shows UseCase3.xml, the Ant script defined for this pattern. In it, you will find four Ant targets or steps, *init*, *Distribute*, *step1* and *job*. The *init* target initializes the properties used in the script. Find the target named *Distribute*. This target uses the Ant task *loadfile* to load the data file distribution list defined by `$`{listName}. It uses the JavaScript program distList1.js to read the distribution list and for each entry in the distribution list creates a transfer request. 
 
 The *Distribute* target also indicates that it will use JavaScript to manipulate the contents of the List1.distributionList file. Within the target, we are using the Ant-Contrib *foreach* command to loop through the contents of the List1.distributionList file running the *step1* target for each record in the file. (The Ant-Contrib project is a collection of tasks for [Apache Ant](http://ant.apache.org/).) 
 
 The *step1* target is the main body of the script, because this is where the transfer request is actually created. *step1* executes another JavaScript file, **distList2.js**. The JavaScript in this file recognizes the pipe delimiter, creates an array, and splits the record into two items in the array. These fields are then used to set the property **TNODE** to the agent and the **dstdir** property to the directory path where the file will be written on the destination. Progress messages are written to the console via the echo.setMessage and echo.perform commands.

#### Using Metadata 

fte:metatdata is a MFT extension to Ant. Metadata is used to carry additional user-defined information with a file transfer operation. fte:Metadata is nested by the following tasks:

 * [fte:filecopy](http://publib.boulder.ibm.com/infocenter/wmqfte/v7r0/topic/com.ibm.wmqfte.doc/file_copy.htm)

 * [fte:filemove](http://publib.boulder.ibm.com/infocenter/wmqfte/v7r0/topic/com.ibm.wmqfte.doc/file_move.htm)

 * [fte:call](http://publib.boulder.ibm.com/infocenter/wmqfte/v7r0/topic/com.ibm.wmqfte.doc/call.htm)

    The fte:entry parameter is specified as a nested element. You must specify at least one entry inside the fte:metadata nested element. You can choose to specify more than one entry. Entries associate a key name with a value. Keys must be unique in a block of fte:metadata. The name and value entry attributes are required. Still referring to target *step1* in Figure 3.5.4, you can see examples of a fte:metadata definition which contains two entries:
    
        <fte:metadata>
        <fte:entry name="mqaDeptName" value="`$`{mqaDeptName}" />
        <fte:entry name="mqaSLAClass" value="`$`{mqaSLAClass}"/>
        <fte:entry name="jobNumber" value="`$`{jobNumber}" />
        <fte:entry name="mqaSLAClass" value="SLADemo"/>
        </fte:metadata>

    The fte:filecopy Ant extension creates a file transfer request for a source file to be copied from the source agent to the destination agent. A fte:filecopy leaves the source file on the source system. A fte:filemove will delete the source file when the transfer is complete.

    fte:filecopy and fte:filemove extensions must include a fte:filespec extension to identify the file to be transferred. A fte:filespec is another nested element just as the fte:metadata. However, the fte:filespec is required, but fte:metadata is not. From *Figure 3.5.4*, here is the example of fte:filespec:
    
        <fte:filespec srcfilespec="${srcfile}" dstdir="${dstdir}" overwrite="true"/>

    Notice that the fte:filespec element identifies the file being transferred and the directory path for storing the file on the destination. It also contains options for the transfer such as the overwrite option to replace an existing file.

### Run Pattern Script 

1.  Run the Ant script with your preferred method.

2.  Monitor runtime console – Console in the toolkit or command prompt window console.

### Review results on MFT Server

1.  Analyze the Transfer Log.

2.  Find proof that the file(s) were transferred.

### Review results using IBM Sterling Control Center 

1.  Follow the steps in **Appendix A** to review your results in IBM Sterling Control Center.

Pattern 4 - Pull files from two spokes of a hub and concatenate into a single file
----------------------------------------------------------------------------------

The pattern demonstrates how to use an Ant transfer template to pull files from two or more end points in asynchronous mode. The Ant script waits for all files to arrive and then concatenates the files into a single file before deleting the temp files.

1.  Pull file asynchronously from AGENT1

2.  Pull file asynchronously from AGENT2

3.  Await for all files to arrive

4.  Concatenate all files into a single file

5.  Delete temp files

    <img src="media/image51.png" width="624" height="224" />

    *Figure 3.6.1 – Pattern 4 - Pull files from two spokes to a hub and concatenate into a single file *

### Reference Material for Pattern 4 

> <img src="media/image52.png" width="525" height="142" />
>
> *Figure 3.6.2 – POT command file for Pattern 4 *

<img src="media/image53.png" width="669" height="460" />

<img src="media/image54.png" width="670" height="456" />

<img src="media/image55.png" width="669" height="458" /><img src="media/image56.png" width="670" height="384" />*Figure 3.6.3 – Ant script for running MFT Pattern 4, UseCase4.xml *

<img src="media/image57.png" width="624" height="464" /><img src="media/image58.png" width="605" height="206" />

*Figure 3.6.4 – hubprocess.xml – command file *

### Review the Ant script UseCase4.xml 

### Using fte:call in Ant Script 

1.  In summary, this pattern is intended to combine two source files from different locations (referred to as spokes) into one file (at the hub). The Ant script “pulls” files from various locations via transfer requests, then uses the fte:call extension to run a concatenate function to combine the files. Referring to Figure 3.6.3, the Ant script appears very long and complex, but if we break it down by the targets, it seems much clearer.

    Starting with the *init* target, initial default property settings are defined. The important properties for this pattern are **spoke1** and **spoke2**. Spoke1 has a value of agent STUDENT1 in the form agent@queuemanager. Spoke2 is set to agent WMQ2008R2DC. **Spoke.file** is the path to the file to be pulled from each spoke. Spoke1 is the student image and spoke2 is the MFT Server machine. The hub is also the student image. But look at the POT Command file (Figure 3.6.2) and you will see that the spokes are being over-ridden by AGENT1 and AGENT2.

    *step1* target will issue a number of commands. fte:filecopy is used twice to create two transfer requests; one from spoke1 (AGENT@MB8QMGR) to hub (STUDENT1@MB8QMGR) and another from spoke2 (AGENT2@MB8QMGR) to hub. Fte:metadata is used for both commands. And each command includes fte:filespec to indicate the source file name/path and destination file name/path. Take note that each source file has the same name, but the first filecopy will add \*.1.txt to the file name and the second filecopy will add \*.2.txt to the file name. Using the substitution variable for hub.file the destination files will be **c:\\Demo\\temp\\hub\_file.1.txt** and **c:\\Demo\\temp\\hub\_file.2.txt**.

    Following the transfer requests, the script will wait for the transfers to complete. The **fte:awaitoutcome** extension to wait on the transfer requests with the IDs specified by spoke1.ID (<STUDENT1@MB8QMGR.ID>) and spoke2.ID (<WMQ2008R2DC@MB8QGMR.ID>).

    1.  ### Using Metadata 

    fte:metatdata is a MFT extension to Ant. Metadata is used to carry additional user-defined information with a file transfer operation. fte:Metadata is nested by the following tasks:

    -   [fte:filecopy](http://publib.boulder.ibm.com/infocenter/wmqfte/v7r0/topic/com.ibm.wmqfte.doc/file_copy.htm)

    -   [fte:filemove](http://publib.boulder.ibm.com/infocenter/wmqfte/v7r0/topic/com.ibm.wmqfte.doc/file_move.htm)

    -   [fte:call](http://publib.boulder.ibm.com/infocenter/wmqfte/v7r0/topic/com.ibm.wmqfte.doc/call.htm)

    The fte:entry parameter is specified as a nested element. You must specify at least one entry inside the fte:metadata nested element. You can choose to specify more than one entry. Entries associate a key name with a value. Keys must be unique in a block of fte:metadata. The name and value entry attributes are required. Still referring to target *step1* in Figure 3.6.4, you can see examples of a fte:metadata definition which contains two entries:

    1.  <img src="media/image59.png" width="458" height="66" />

    The fte:filecopy Ant extension creates a file transfer request for a source file to be copied from the source agent to the destination agent. A fte:filecopy leaves the source file on the source system. A fte:filemove will delete the source file when the transfer is complete.

    fte:filecopy and fte:filemove extensions must include a fte:filespec extension to identify the file to be transferred. A fte:filespec is another nested element just as the fte:metadata. However, the fte:filespec is required, but fte:metadata is not. From *Figure 3.6.4*, here is the example of fte:filespec:

    1.  <img src="media/image60.png" width="602" height="17" />

    Notice that the fte:filespec element identifies the file being transferred and the file name for storing the file on the destination. It also contains options for the transfer such as the overwrite option to replace an existing file.

### Run Pattern Script 

1.  Run the Ant script with your preferred method.

2.  Monitor runtime console – Console in the toolkit or command prompt window console.

### Review results on MFT Server

1.  Analyze the Transfer Log.

2.  Find proof that the file(s) were transferred.

### Review results using IBM Sterling Control Center 

1.  Follow the steps in **Appendix A** to review your results in IBM Sterling Control Center.

Pattern 5 - Use PGP with MFT
----------------------------

This pattern demonstrates how to configure an Ant transfer template to perform a PGP encrypt using a “Pre Source Call”, then using a “Post Dest Call” to decrypt the file using PGP. The example shows how one can pass in the PGP key information such as Ant properties.

1.  preSrc Call to Encrypt file before transfer

2.  File Transfer of Encrypted file

3.  postDst Call to Decrypt file

<!-- -->

1.  <img src="media/image61.png" width="624" height="204" />

    *Figure 3.7.1 – Pattern 5 – Use PGP with MFT *

You can run programs on a system where a IBM MQ Managed File Transfer agent is running. As part of a file transfer request, you can specify a program to run either before a transfer starts, or after it finishes. Additionally, you can start a program that is not part of a file transfer request by submitting a managed call request.

There are five scenarios in which you can specify a program to run:

-   As part of a transfer request, at the source agent, before the transfer starts

-   As part of a transfer request, at the destination agent, before the transfer starts

-   As part of a transfer request, at the source agent, after the transfer completes

-   As part of a transfer request, at the destination agent, after the transfer completes

-   Not as part of a transfer request. You can submit a request to an agent to run a program. This scenario is sometimes referred to as a managed call.

Programs can be started using one of five nested elements: fte:presrc, fte:predst, fte:postdst, fte:postsrc, and fte:command. These nested elements instruct an agent to call an external program as part of its processing. Before you can start a program, you must ensure that the command is in a location specified by the commandPath property in the agent.properties file of the agent that runs the command.

### Reference Material for Pattern 5

> <img src="media/image62.png" width="569" height="160" />

1.  *Figure 3.7.2 – POT command file for Pattern 5 *

    <img src="media/image63.png" width="654" height="392" /><img src="media/image64.png" width="654" height="437" />

    *Figure 3.7.3– Ant Script for running Pattern 5, UseCase5.xml*

### Review the Ant script UseCase5.xml 

1.  Referring to Figure 3.7.2 and 3.6.3, you see four targets in project UseCase5, *init*, *step1*, *check1*, and *job*. Each target is dependent on the previous one, except *job* which is dependent on all previous targets. If *job* runs, that means all prior targets were successful and we have a good end-of-job.

    As you can see there are numerous properties initialized in the *init* target. A number of these are over-ridden in the command file (Figure 3.7.2). There are nine properties over-ridden in the command file. Actually they may be specified explicitly even though they are the same as the default values. Please identify the source and target agents as well as the source and destination files.

    In *step1* this pattern will transfer a file with the fte:filecopy task. The source and destination agent names are being substituted with the names defined in *init*. The fte:filespec is a nested element and specifies the names of the source and destination file names. The names are being substituted with the values defined in *init*. Notice that the file name is built in parts with the base name and extension. fte:presrc and fte:postdst are also nested elements. fte-presrc specifies a program invocation to take place at the source agent before the transfer starts. fte-postdst specifies a program invocation to take place at the destination agent after the transfer has completed. Arguments for the programs specified by fte:presrc and fte:postdst are indicated by the pre:arg

    After close analysis of the tasks and nested elements you will see that file C:\\Demo\\disk1\\file8.txt will be encrypted by the source agent STUDENT1 before being transferred. It will be encrypted by the gpg.exe program located in the C:\\Demo\\pgp directory with key with the label “<hbhyat@us.ibm.com>” located in the same directory. The intermediate file which gets transferred is file8.gpg. On agent WMQ2008R2DC, after the transfer is complete, the file8.gpg file gets decrypted by the command decrypt.cmd.

### Run Pattern Script 

1.  Run the Ant script with your preferred method.

2.  Monitor runtime console – Console in the toolkit or command prompt window console.

### Review results on MFT Server

1.  Analyze the Transfer Log.

2.  Find proof that the file(s) were transferred.

### Review results using IBM Sterling Control Center 

1.  Follow the steps in **Appendix A** to review your results in IBM Sterling Control Center.

Pattern 6 - Dynamically set job name and MFT metadata dynamically based on file contents
----------------------------------------------------------------------------------------

So far you have seen how to use MFT extensions to call programs and set metadata. This pattern demonstrates how to use JScript within an Ant script and the E4X feature to introspect a file before sending it. During the introspection, data elements are extracted and used as MFT metadata values of the transfer. These values are then used to dynamically set the Job Name metadata of the transfer. This makes it much easier to find transfers in the MFT transfer log.

> <img src="media/image65.png" width="624" height="210" />

*Figure 3.8.1 – Pattern 6 - Dynamically set job name and MFT metadata based on file contents *

|                                                                       |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|-----------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <img src="media/image1.png" alt="sign-info" width="57" height="57" /> | Information                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
  **ECMAScript for XML (E4X) is programming language extenstion** that adds native XML support to ECMAScript (which includes ActionScript, JavaScript, and JScript). The goal is to provide an alternative to DOM interfaces that use a simpler syntax for accessing XML documents. It also offers a new way of making XML visible. Before the release of E4X, XML was always accessed at an object level. E4X instead treats XML as a primitive (like characters, integers, and booleans). This implies faster access, better support, and acceptance as a building block (data structure) of a program.  
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
  E4X is standardized by Ecma Internationsl in the ECMA-357 standard. The first edition was published in June 2004, the second edition in December 2005.                                                                                                                                                                                                                                                                                                                                                                                                                                                   
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           
  .                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |

### Reference Material for Pattern 6 

> <img src="media/image66.png" width="580" height="81" />
>
> *Figure 3.8.2 – POT command file for Pattern 6 *

<img src="media/image67.png" width="629" height="418" /><img src="media/image68.png" width="678" height="401" />

*Figure 3.8.3 – Ant script for running MFT Pattern 6, UseCase6.xml *

<img src="media/image69.png" width="462" height="481" /> <img src="media/image70.png" width="440" height="305" />

*Figure 3.8.4 – PO\_FILE1.txt – data file *

### Review the Ant script UseCase6.xml 

### Using alternate programming constructs 

1.  In summary, this pattern loads the data file to be transferred and extracts certain information from the data to be sent as MFT metadata of the transfer. The Job Name property is declared in the Ant script then populated with the purchase order number from the file. This metadata travels with the file so it can easily be tracked by the PO number. This is quite helpful for business analysts and operations. It also provides an audit point to prove purchase orders were received and / or processed.

    Referring to the UseCase6.xml opened in Eclipse or Figure 3.8.3, locate line 31. Line 31 defines a property named “PO” and loads the file defined by the ${srcfile} substitution variable. On line 24, you see that ${srcfile} resolves to C:\\Demo\\disk1\\PO\_FILE1.xml. This is the file which is to be transferred and contains the business information “PO number”.

    Locate the &lt;script&gt;&lt;/script&gt; stanza on line 32. As you can see, the language is JavaScript. Line 33 defines a variable called strData which points to the property “PO” (file PO\_FILE1.txt). Line 34 creates an object of the file using E4X . Now that we have an object, lines 35 and 36 are creating properties “ORIGINALPONUM” and “ITNPONum” with the setPorperty command and abstracting the XML elements of the same names from the file PO\_FILE1.txt shown in Figure 3.8.4, lines 20 and 21. Line 37 in the Ant script then sets the property “jobName” with the value “PO\_Number\_ “ and concatenates that with the abstracted value of the ORIGINALPONUM.

> <img src="media/image71.png" width="662" height="264" />
>
> The target *step1* then creates a transfer request with the fte:filecopy extension and names the transfer with the jobname which now includes the PO number. As we have seen in other transfers, other metadata is included with the transfer, one of which is the PO number.
>
> <img src="media/image72.png" width="624" height="169" />

### Run Pattern Script 

1.  Run the Ant script with your preferred method.

2.  Monitor runtime console – Console in the toolkit or command prompt window console.

### Review results on MFT Server

1.  Analyze the Transfer Log.

2.  Find proof that the file(s) were transferred.

### Review results using IBM Sterling Control Center 

1.  Follow the steps in **Appendix A** to review your results in IBM Sterling Control Center.

Pattern 7 - Use a Custom Ant task to validate the contents of a file 
---------------------------------------------------------------------

Pattern 7 demonstrates what is possible with custom Ant directives. In this use case we show how a custom Ant directive can be used to check the contents of a file which is about to be sent. Normally one could certainly use JScript to read and validate the contents of a file, but in this case the file contains EBCDIC data including packed decimal elements. These elements must be added together and checked against a threshold value which is passed in as Ant properties to the transfer request. If the values in the file exceed the set thresholds the transfer should not continue. The source for the custom Ant directive is shown in Figure 3.9.4 - *validate\_CommFile.xml*.

1.  <img src="media/image73.png" width="624" height="204" />

    *Figure 3.9.1 – Pattern 7 - Use a custom Ant task to validate the file contents *

### Reference Material for this exercise 

> <img src="media/image74.png" width="574" height="193" />
>
> *Figure 3.9.2 – POT command file for Pattern 7 *

<img src="media/image75.png" width="673" height="432" /><img src="media/image76.png" width="674" height="412" /><img src="media/image77.png" width="673" height="69" />

*Figure 3.9.3 – Ant script for running MFT Pattern 7, UseCase7.xml *

<img src="media/image78.png" width="682" height="281" />

*Figure 3.9.4 – validate\_CommFile.xml – Ant script *

### Review the Ant script UseCase7.xml 

### Using alternate programming constructs 

1.  Referring to Figure 3.9.3 or the file UseCase7.xml in the toolkit, you can observe that the Ant target *copy* depends on the target *init* and uses the fte:filecopy command to request a transfer. The source agent is STUDENT1 because it is the default value as assigned in the *init* target. Referring to Figure 3.9.2 or the POTCommand file UseCase7.cmd in the Notepad++ editor, you see the SNODE property is not being over-ridden, but the DNODE is and resolves to WMQ2008R2DC.

    Looking at lines 54 – 58 of the UseCase7.xml, you see the nested element fte:presrc of the fte:copy command. This specifies the program (Ant directive) which will be called on the source node prior to the transfer. The command is another Ant script named validate\_CommFile.xml which is shown in Figure 3.9.4.

    <img src="media/image79.png" width="624" height="65" />

    The validate\_CommFile.xml script will check that the values within the file being transferred do not exceed the supplied thresholds. If it does then the transfer is aborted. Output is generated to the console that becomes part of the transfers audit log which captures the values and record count within the file. In order to accomplish the script executes a java program named countCommission. The program name, classname, and classpath are specified in the *init* target with a **taskdef** property.

    <img src="media/image80.png" width="660" height="58" />

    The *calculate* target is where the program actually gets executed. countCommission counts the number of characters and the number of records in the file specified in $(srcfile). If either exceeds the threshold properties specified in the *init* of UseCase7.xml, then it sets the boolean property **value\_threshold\_exceeded** which is checked in the *fail* target.

    <img src="media/image81.png" width="670" height="137" />

### Run Pattern Script 

1.  Run the Ant script with your preferred method.

2.  Monitor runtime console – Console in the toolkit or command prompt window console.

3.  Change the threshold values to 1000 for **totalValue\_threashold** or 10 for **recordCount\_threashold** and run the pattern again. (Apologies for the developer who doesn’t know how to spell threshold.)

### Review results on MFT Server

1.  Analyze the Transfer Log.

2.  Find proof that the file(s) were transferred.

### Review results using IBM Sterling Control Center 

1.  Follow the steps in **Appendix A** to review your results in IBM Sterling Control Center.

Pattern 8 - Transfer single file to multiple end points using RegEx to dynamically build the distribution list 
---------------------------------------------------------------------------------------------------------------

As an alternative to the distribution list in Pattern 3 (section 3.5), we can use an Ant Regular Expression to dynamically build a distribution list then use the list to distribute a single file.

The powerful Ant RegExp function is used to select MFT agent names (stored as zero byte files) from a named folder and then reformat the names into a list. The list is then used in a “for each” Ant construct which transfers the file to each node in the list.

The purpose is to show the flexibility and power of the Ant automation.

> <img src="media/image82.png" width="622" height="225" />

*Figure 3.10.1 – Pattern 10 - Transfer a single file to multiple endpoints using RegEx to dynamically build the distribution list *

In computing, a **regular expression** is a specific pattern that provides concise and flexible means to "match" (specify and recognize) strings of text, such as particular characters, words, or patterns of characters. Common abbreviations for "regular expression" include **regex** and **regexp**.

The concept of regular expressions was first popularized by utilities provided with [Unix](http://en.wikipedia.org/wiki/Unix) distributions, in particular the editor *ed* and the filter *grep*. A regular expression provides a grammar for a formal language; this specification can be interpreted by a regular expression processor, which is a program that either serves as a parser generator or examines text and identifies substrings that are members of the specified (again, formal) language. Historically, the concept of regular expressions is associated with Kleene’s formalism of regular sets introduced in the 1950s.

Regular expressions are used by many text editors, utilities, and programming languages to search and manipulate text based on patterns. Some of these languages, including Perl, Ruby, AWK, and TCL, integrate regular expressions into the syntax of the core language itself. Other programming languages like .NET languages, Java, Pyhthon, and C++ (since C++11) instead provide regular expressions through standard libraries. For many other languages, such as Object Pascal (Delphi), C, and earlier versions of C++, non-core libraries are available.

Many modern computing systems provide wildcard characters in matching filenames from a file system. This is a core capability of many command-line shells and is also known as globbing. Wildcards differ from regular expressions in generally expressing only limited forms of patterns.

IBM MQ Managed File Transfer uses regular expressions in a number of scenarios. For example, regular expressions are used to match user IDs for Connect:Direct® security credentials, to split a file into multiple messages by creating a new message each time a regular expression is matched, and to specify a set of files to transfer in a file transfer request. The regular expression syntax used by IBM MQ Managed File Transfer is the syntax supported by the java.util.regex API. This regular expression syntax is similar to, but not the same as, the regular expression syntax used by the Perl language.

Here are some examples of using regular expressions:

-   To match all patterns, use the following regular expression:

    .\*

-   To match all patterns that begin with the string fte, use the following regular expression:

    fte.\*

-   To match all patterns that begin with the string accounts followed by a single digit, and end with .txt, use the following regular expression:

    accounts\[0-9\]\\.txt

### Reference Material for Pattern 8 

> <img src="media/image83.png" width="477" height="122" />
>
> *Figure 3.10.2 – POT command file for Pattern 8 *
>
> <img src="media/image84.png" width="690" height="444" /><img src="media/image85.png" width="685" height="232" />

*Figure 3.10.3 – Ant script for running MFT Pattern 8, UseCase8.xml *

<img src="media/image86.png" width="658" height="160" />

*Figure 3.10.4 – Zero byte files representing agents in distribution list *

### Review the Ant script UseCase8.xml 

### Building distribution lists using regex 

1.  This pattern offers an alternative to Pattern 3 (section 3.5). It shows another way to create a distribution list of multiple endpoints. A directory of zero-byte files was created to store the agents which make up the distribution list. As shown in Figure 3.10.4, the path C:\\Demo\\loop\\list1 is a directory which contains three zero-byte files. The files use the naming pattern of agent@queuemanager.

    The target *loop* in UseCase8.xml repeats for each file in the directory a RegEx for any characters in front of the file name and selects only the filename to move into the property *target.agent.name*. Referring to Figure 3.10.3, line 31 – 33 id uses the fileset command to point to the directory C:/Demo/loop/list1 including all files in the directory. The &lt;sequential&gt;&lt;/sequential&gt; stanza (lines 34 – 49) is performed **for** each file in the directory per the &lt;for&gt; &lt;/for&gt; stanza. Therefore for each file in the directory, the file name is used as the destination agent with the RegEx (line 36 – 37) and a transfer request is created with the fte:filecopy extension (line 38 – 39).

### Run Pattern Script 

1.  Run the Ant script with your preferred method.

2.  Monitor runtime console – Console in the toolkit or command prompt window console.

### Review results on MFT Server

1.  Analyze the Transfer Log.

2.  Find proof that the file(s) were transferred.

### Review results using IBM Sterling Control Center 

1.  Follow the steps in **Appendix A** to review your results in IBM Sterling Control Center.

Pattern 9 - Split “STACK File” using JScript and then programmatically invoke Ant to transfer each generated file 
------------------------------------------------------------------------------------------------------------------

The main purpose of this pattern is to show how Ant jobs can be kicked off programmatically using JScript while also showing how JScript can be used to create files.

This pattern demonstrates how one can use JScript to split a stack file. In this case a stack file is a file which contains data for several stores separated by header / trailer markers. This is very common in the retail industry. It shows how one can use JScript within an Ant script to read the contents of a stack file and split it into separate mini files one for each header / trailer pair. After JScript builds the “mini” files, it invokes an Ant job to programmatically transfer the files to the target agents based on the content of the header record.

This example also demonstrates how JScript can be used to dynamically invoke the Ant template Split\_Transfer1.xml shown in Figure 3.11.4 and pass in properties. We will review the logic of the JScript SplitFile.js shown in Figure 3.11.5.

> <img src="media/image87.png" width="624" height="206" />
>
> *Figure 3.11.1 – Pattern 9 - Split stack file using JScript and invoking Ant to generate transfer requests *

### Reference Material for Pattern 9 

> <img src="media/image88.png" width="504" height="92" />
>
> *Figure 3.11.2 – POT command file for Pattern 9 *
>
> <img src="media/image89.png" width="680" height="442" />
>
> *Figure 3.11.3 – Ant script for running MFT Pattern 9, UseCase9.xml *
>
> <img src="media/image90.png" width="669" height="517" /><img src="media/image91.png" width="614" height="184" />
>
> *Figure 3.11.4 – Split\_Transfer1.xml file for splitting stack file *

<img src="media/image92.png" width="678" height="413" /><img src="media/image93.png" width="677" height="430" />

<img src="media/image94.png" width="674" height="446" /><img src="media/image95.png" width="673" height="161" />

*Figure 3.11.5 – Split\_Transfer1\`.xml file for splitting stack file*

### Review the Ant script UseCase13.xml

### Using JScript to create files and invoke Ant scripts 

1.  Referring to Figure 3.11.2, we can see this is the command file used to start this pattern. It is only calling *fteAnt* for UseCase9.xml and giving it a job name. No other properties are overridden.

    Referring to UseCase9.xml Ant script (Figure 3.11.3), the only thing significant in this script is that it is running JavaScript (Line 27) and the script file is **C:\\IBM\\fteAnt\\Demos\\JavaScripts\\SplitFile.js**. However, the other properties are also notable. Especially the property **fteAntJob** which points to the **C:\\IBM\\fteAnt\\Demos\\DemoUseCases\\Split\_Transfer1.xml** shown in Figure 3.11.4. This should look familiar by now. It is basically another transfer request using the fte: extensions. What is interesting is that this is the XML file being invoked by the JavaScript to transfer each of the mini files which it previously split from the stack file. srcfile and tempPath properties are identifying the stack file and the path to store the mini files once they are transferred.

    Since the Ant script calls the JScript SplitFile.js, let’s refer to Figure 3.11.5 or the opened file in Notepad++. Lines 1 – 24 are comments which explain the format of the file and present an example file. Review the comments now. Lines 27 – 36 are import statements for the necessary classes. Lines 38 – 40 create an Ant task and display a message on the console. Lines 43 and 44 are getting properties from the calling Ant script. Lines 45 – 54 are defining variables to be used by the script. Lines 45 – 47 are particularly important because they are defining storage areas for the file records input, buffer, and output streams.

    The rest is the basic program logic with programming constructs do-while, if-else, and a function. The code in the while stanza will be executed as long as the length of the data input stream is not zero (Line 55). Lines 57 – 65, read a record from the file (data input stream) and check the length to see if it is a header record (length = 25). If it is, it means we are starting a new “mini” file. Since it is a new file, we check to see if an output file is open by testing *outf* for not null. If it is not null (outf != null), we close the output file, display the records written, then perform the function runAnt (Lines 93 – 113).

    The store Number, dir, and file name variables are passed when runAnt is called. The Ant script which is performed was defined by the property name fteAntJob in UseCase9.xml (Figure 3.11.3, Line 26) with default value Split\_Transfer1.xml. Lines 95 – 104 of the function runAnt are setting the properties for Split\_Transfer1.xml. The storeNumber variable is being used to create some of those properties by appending it with some text, for instance line 95 and line 101.

    Lines 106 – 112 adds a helper project and references, then executes the buildFile.

    Returning to the do / while loop, the new file is created for the next store in the header record, the file is opened, and a new output stream is created. Count is reset, record count (fcount) is incremented. The loop continues and each out record is written to the output stream until another header record is encountered, in which case runAnt is called again for another “mini” file.

    As noted above, runAnt builds properties for Split\_Transfer1.xml and executes that Ant script. Referring to Figure 3.11.4, you will see three targets in Split\_Transfer1.xml – *init*, *step1*, and *check1*. After reviewing the previous patterns, these should look familiar by now. *Init* defines properties. Notice that a number of the properties have a null value (ex. Line 11). This is because those properties are being passed by the JavaScript Split\_File.js discussed above. *Step1* is creating a file transfer request with the fte:filemove extension. This is the same as the fte:filecopy extension, except the source file is being moved, that it is transferred, then deleted from the source. The parameters are source and destination agents (src and dst) and outcome=”await” which means Ant will wait until the transfer is complete before continuing. Fte:metadata is included for tracking purposes. Fte:filemove requires a fte:filespec extension to define the source and destination files. Those also were passed from the JavaScript. The overwrite parm is set to true which tells the destination agent to replace an exisitng file. *Check1* target ensures successful return codes or fails the build if there was a problem with the transfer. Each target is dependent on the previous targets, so conditional processing is enabled.

### Run Pattern Script 

1.  Run the Ant script with your preferred method.

2.  Monitor runtime console – Console in the toolkit or command prompt window console.

### Review results on MFT Server

1.  Analyze the Transfer Log.

2.  Find proof that the file(s) were transferred.

### Review results using IBM Sterling Control Center 

1.  Follow the steps in **Appendix A** to review your results in IBM Sterling Control Center.

Pattern 10 - Invoke a remote Ant job to have it pull a file 
------------------------------------------------------------

While a typical use case either pushes or pulls a file to or from a remote agent, sometimes the use case requires an ability to call the remote agent to have it “come and get” a named file so that it can control the name of the destination file and location.

This use cases demonstrates how this can be accomplished using the “manage call” capabilities of MFT.

1.  Submit a manage call to TEACHER agent asking it to invoke Ant template Transfer PatternA.xml

2.  Agent TEACHER invokes transfer PatternA.xml passing on the properties passed by the requestor

3.  Transfer PatternA.xml pulls the named file from agent STUDENT1 and assigns a unique destination file name

<img src="media/image96.png" width="624" height="204" />

> *Figure 3.11.1 – Pattern 10 – Pull Transfer *

### Reference Material for Pattern 10 

> <img src="media/image97.png" width="450" height="86" />
>
> *Figure 3.12.2 – Ant script for running MFT Pattern 10 *
>
> <img src="media/image98.png" width="693" height="393" /><img src="media/image99.png" width="684" height="430" />
>
> *Figure 3.12.3 – Ant script for running Pattern 10, UseCase10.xml*

### Review the Ant script UseCase10.xml 

1.  Referring to the command file, Figure 3.12.2:

    -   The first command sets the path to the correct directory.

    -   The next command is a call to fteAnt. This is the same as typing the fteAnt command on the command line with the –f parameter which indicates the Ant script.

    -   The –D is a java convention for overriding parameters. In this case, the jobname is over-ridden with **STUDENT1\_UseCase10** and the srcfile is over-ridden with **C:\\Demo\\disk1\\File4.txt**.

    Notice the steps (targets or tasks) of the script and the fte: commands that are being issued by the script.

    -   You can see that the script executed multiple steps to complete the transfer.

    -   Step “init” is our housekeeping step that assigns values to variables. The default values are defined here and can be over-ridden at runtime. Important values here are SNODE and TNODE and callName.

    -   “step1 is dependent on “init” and executes a fte:call on the agent specifiied by the TNODE property which is WMQ2008R2DC@MB8QMGR. The convention followed here is &lt;agentname@qmgr&gt;. This tells us the agent that will do the call is WMQ2008R2DC and the agent’s queue manager is MB8QMGR which is running on the Teacher machine – Windows 2008 server. We are running the UseCase10 Ant script on the Student machine, but the call is being made to another Ant script – UseCaseA indicated by the callName parameter on the fte:command. We must review UseCaseA on the Teacher machine to see what will happen when the agent WMQ2008R2DC runs it.

    -   “check1” is dependent on “step1” and checks return codes to either fail the script or let it continue.

    Review the targets, commands and properties.

    This script has four targets:

    -   Init - initialization

    -   step1 – main processing, dependent on init

    -   check 1 – checks return codes, dependent on step1

    -   job – successful script processing, dependent on all previous targets

    Using the code snippet above, locate the code which is performing the following:

    -   step1, as you see by the description, is pulling the remote file.

    -   It will execute a fte:filecopy command which is a transfer without deleting the source file. The file will be copied from the source agent specified by the SNODE variable to the destination agent specified by the TNODE variable.

    -   fte:filecopy is the command, but the fte:filespec tag specifies the file to be copied. srcfilespec is the element for the name of the source file specified by the srcfile variable and dstfilespec is the element for the name of the destination file specified by the dstfile variable.

        The init target defines the default values for the variables used in the script. Only the dstfile variable has a default value. The rest of the variables have not been given values. Even the dstfile value has variables within the name which need to be resolved. So where are the values coming from?

        Ans: The values for those variables are passed from UseCase10.xml script which will be submitted from the Student machine.

    -   The fte:metadata stanza defines name – value pairs that are carried on the transfer and are user defined.

### Run Pattern Script 

1.  Run the Ant script with your preferred method.

2.  Monitor runtime console – Console in the toolkit or command prompt window console.

### Review results on MFT Server

1.  Analyze the Transfer Log.

2.  Find proof that the file(s) were transferred.

### Review results using IBM Sterling Control Center 

1.  Follow the steps in **Appendix A** to review your results in IBM Sterling Control Center.

Pattern 11 - Unzip two files each containing multiple files, concatenate all extracted files into a single file and send to remote agent
----------------------------------------------------------------------------------------------------------------------------------------

Pattern 11 demonstrates how to call an Ant template at the transfer’s preSrc call point. The called template UnZip.Transfer1.xml uses the built-in Ant unzip capabilities to unzip two files. It then concatenates all the extracted files found within both zip files into a single file and transfers this one file to a remote agent.

1.  Unzip file 1 to temp directory.

2.  Unzip file 2 to temp directory.

3.  For each “extracted file” in temp directory, concatenate into a dynamically name target file

4.  Move dynamically created file to remote agent

5.  Delete temp files

    <img src="media/image100.png" width="624" height="213" />

    *Figure 3.13.1 – Pattern 11 – Unzip two files ech containing multiple files, concatenate all extracted files into a single file and send to remote agent *

### Reference Material for Pattern 11

> <img src="media/image101.png" width="234" height="62" />
>
> *Figure 3.13.2 – POT command file for Pattern 11 *
>
> <img src="media/image102.png" width="629" height="440" /><img src="media/image103.png" width="637" height="356" />
>
> *Figure 3.13.3 – Ant script for running MFT Pattern 11, UseCase11.xml *
>
> <img src="media/image104.png" width="624" height="429" /> <img src="media/image105.png" width="624" height="208" />
>
> *Figure 3.13.4 – Ant script for running MFT Pattern 11 *
>
> <img src="media/image106.png" width="596" height="446" />
>
> *Figure 3.13.5 – Java calcSpace.js for calculating space *

### Review the Ant script UseCase11.xml 

1.  Referring to Figure 3.13.2, we can see this is the command file used to start this pattern. It is only calling *fteAnt* for UseCase11.xml (Figure 3.13.3) and giving it a job name. No other properties are overridden.

    However, UseCase11.xml uses a JavaScript (Line 42) “**C:\\IBM\\fteAnt\\Demos\\JavaScripts\\calcSpace.js**” which is used to calculate the space for the target file. You can view the JavaScript in Figure 3.13.5. This was originally used for calculating space on a destination z/OS file system. However, for demonstration purposes, we left the script the same even though we are running on Windows.

    Notice that the UseCase11.xml script only has three targets – *init* for initializing properties and *job* for completing the dependencies. As mentioned at the beginning of this pattern, UseCase11.xml will call another Ant Template “UnZip.Transfer1.xml” (Figure 3.13.4). A pre-source (fte:presrc) call point within the fte:filecopy command is used to call UnZip.Transfer1.xml. UnZip. The fte:presrc nested element also defines metadata properties which will be used by UnZip.Transfer1.xml.

    Transfer1.xml executes the Ant directive **unzip**. Notice that there are no fte:xxxxxx commands nor nested elements in this script. It is strictly native Ant. After UnZip.Transfer1.xml unzips both files, it then concatenates all the unzipped files into one file which will be transferred by the fte:filecopy command.

    The *copy* target is the main processing of the script where it executes the fte:filecopy command. You also see a condition stanza for checking the return code of the fte:filecopy. Looking more closely at the fte:filecopy, you will notice several metadata properties defined for the copy. The fte:filecopy specifies the source, destination, priority, jobname, and the return code property. On line 57, you will see the nested element fte:filespec for the source file and destination file names.

### Run Pattern Script 

1.  Run the Ant script with your preferred method.

2.  Monitor runtime console – Console in the toolkit or command prompt window console.

### Review results on MFT Server

1.  Analyze the Transfer Log.

2.  Find proof that the file(s) were transferred.

### Review results using IBM Sterling Control Center 

1.  Follow the steps in **Appendix A** to review your results in IBM Sterling Control Center.

Pattern 12 - Transform CSV files to XML using App Connect Enterprise 
----------------------------------------------------------------------

This pattern demonstrates how comma separated values (CSV) files can be sent directly to a IBM App Connect Enterprise (ACE) message flow for transformation and routing using IBM MQ Managed File Transfer. Beginning with WebSphere Message Broker V8, ACE includes Managed File Transfer (FTE) Nodes for sending and receiving files via MFT. If these nodes are included in a deployed message flow, WMB starts an embedded MFT agent to participate in MFT transfers.

The FTE Nodes receive the files including MFT metadata which specifies where the transformed XML files are to be returned. This use case demonstrates the ability for a FTE Node to receive the file, pass it to an ESQL Compute node to transform the data to XML. The compute node also dynamically accesses the MFT metadata which indicates where the transformed data is to be returned. The new file is then transferred via MFT by the FTE Output Node to destination which was carried in the metadata.

1.  <img src="media/image107.png" width="646" height="216" />

    *Figure 3.14.1 – Pattern 12 – Transform CSV files to XML using WMB *

### Reference Material for Pattern 12

> <img src="media/image108.png" width="366" height="466" />

1.  *Figure 3.14.2 – POT Command file for Pattern 12 *

> <img src="media/image109.png" width="657" height="336" /><img src="media/image110.png" width="657" height="240" />

1.  *Figure 3.14.3 – POT Command file for Pattern 12, UseCase12.xml*

> <img src="media/image111.png" width="652" height="402" />

1.  *Figure 3.14.4 – WMB Message Flow with FTE Nodes*

    <img src="media/image112.png" width="624" height="381" /> <img src="media/image113.png" width="694" height="408" />

    *Figure 3.14.5– ESQL of CSV2XML compute node for transforming CSV to XML *

### Review the Ant script UseCase12.xml 

1.  Figure 3.14.1 shows that WMB is running on the MFT Server (teacher machine). As with the previous patterns, we are submitting Ant scripts on the MFT Client (student machine) to transfer files from the MFT Client to the MFT Server and in some cases transferring files back from the server to the client machine. If you are familiar with the WMB toolkit and want to review the message flow and the node properties, you must start the WMB toolkit on the **teacher** machine.

    Referring to Figure 3.14.2, we can see this command file is calling *fteAnt* multiple times, each time running the same Ant script. **MB8BROKER.default** is the destination agent for each call. This is the name of the embedded MFT agent running in WMB. The standard name is formed by concatenating the broker name and execution group name where the agent is running and separating them with a period. Remember that the agent can be qualified with its queue manager name to form the node name. The fully qualified destination node name is **MB8BROKER.default@MB8QMGR**. The destination directory name is also the same for each call. But each call is going to transfer different source files.

    Figure 3.14.3 shows the script, UseCase12.xml, that each *fteAnt* call is running. Each call is overriding the source file name with a different value. The rest of the nested elements and properties should look familiar by now.

    Referring to Figure 3.14.4, there are five FTE Input nodes in the message flow. Each one is monitoring the destination directory C:\\Demo\\WMQ\_In. However, each node is “watching” for a different file name. FTE Input1 is watching for \*CSV\_msg1\*.csv and FTE Input2 is watching for \*CSV\_msg2\*.csv and so forth. The FTE input nodes read the CSV files and parses them into logical message trees according to a model stored in the broker. The logical messages are then passed to the CSV2XML compute node which maps the message tree to a XML model also defined in the broker.

    Figure 3.14.5 shows the ESQL code running in the compute node. The metadata passed with the initial transfer was retained in the local environment of the logical message tree and is now used to build the output XML file name and the destination agent for the return transfer.

### Run Pattern Script 

1.  Run the Ant script with your preferred method.

2.  Monitor runtime console – Console in the toolkit or command prompt window console.

### Review results on MFT Server

1.  Analyze the Transfer Log.

2.  Find proof that the file(s) were transferred.

### Review results using IBM Sterling Control Center 

1.  Follow the steps in **Appendix A** to review your results in IBM Sterling Control Center.

Pattern 13 - Web Gateway 
-------------------------

This pattern shows how to use the IBM MQ Managed File Transfer Service Web Gateway to transfer files to IBM MQ Managed File Transfer (MQMFT) agents and retrieve the status of transfers using an HTTP client.

You can use the Web Gateway to extend an existing IBM MQ Managed File Transfer network to support clients that use the HTTP protocol. The Web Gateway provides a link from clients that are using the HTTP protocol into a IBM MQ Managed File Transfer network that already exists. Transfers that use the Web Gateway are logged throughout the transfer.

> <img src="media/image114.png" width="594" height="298" />
>
> *Figure 3.15.1 – Pattern 13 - Web Gateway *

The Web Gateway application requires a Java™ Platform, Enterprise Edition 5-compliant application server which is not provided with IBM MQ Managed File Transfer. This application server hosts the Web Gateway application. HTTP requests from clients are directed to the application server, which passes the contents of the requests to the application.

A Web Gateway consists of several parts:

-   The MQMFT Web Gateway application

    The Web Gateway application handles both file uploads and transfer status requests.

    When a file is uploaded, the Web Gateway application writes the file data to a temporary store on the file system of the system that the application is running on. The Web Gateway application then submits a file transfer request to the MQMFT agent, which is running on the same system. For more information on this request.

    When a request for status information is received, the Web Gateway application connects to the MQMFT database logger database (using the data access facilities provided by the application server) to retrieve the required information. The application then generates the response, which is passed to the client.

-   An MQMFT web agent

    The Web Gateway requires an MQMFT agent installed on the same system as the application. This agent receives the file transfer request message described in the previous section. The request message refers to the file or files in the temporary store. The agent transfers the files to an existing agent in the MQMFT network, reading the files from the file system store. The source disposition behavior is set to delete so that the files are removed after the transfer successfully completes.

    You do not need to specially configure this agent, because the file transfer request is an ordinary message and not specific to the Web Gateway.

-   The MQMFT database logger and a supported database

    To provide status information about transfers, started using the web or by other means, the Web Gateway application must be able to query a database that contains audit information for MQMFT activity. This database is populated by the database logger component provided with the product. Database access is provided by the data access facilities included in each application server. The database does not need to be located on the same system as the other components.

The Web Gateway is useful if you have files on a system where you do not want to run an agent but where you can use an HTTP client. For example, you can use the Web Gateway for the following tasks:

-   Sending files to a IBM MQ Managed File Transfer agent from a web page

-   Monitoring the status of transfers from a web page

-   Sending files from a portable device that is not capable of running the IBM MQ Managed File Transfer infrastructure but has HTTP capabilities

-   Sending files from an operating system that the IBM MQ Managed File Transfer agent is not supported on

The Web Gateway is used to make files available to users in file spaces. A file space is a reserved area of file storage that is associated with a Web Gateway user. A user who owns a file space can download files at their own convenience, and they do not need an agent or other IBM MQ Managed File Transfer infrastructure to download the file.

In this lab, you will connect to the teacher’s computer over HTTP and send / receive files from a file space on the Web Gateway.

### Reference Material for Pattern 13 

> <img src="media/image115.png" width="649" height="393" />
>
> *Figure 3.15.2 – Pattern 13 - Web Gateway *

### Review Pattern 13 – Web Gateway 

1.  Referring the Figure 3.15.2, the sequence of events follows:

<!-- -->

1.  The user, or a process, sends a file transfer request (in the form of a IBM MQ message) into the IBM MQ network. This request can be sent from the command line or through another WMQ MFT interface. The message is addressed to the queue manager to which the agent on the source system is connected.

2.  The agent on the source system receives the message, which instructs it to perform a file transfer to the web agent.

3.  The agent reads the file from the source file system and converts it to a sequence of IBM MQ messages.

4.  The agent sends the sequence of messages to a queue manager in the IBM MQ network.

5.  The IBM MQ network routes the messages, which contain the file data, to the agent queue manager.

6.  The web agent receives the messages, which contain the file data, from the agent queue manager.

7.  The web agent writes the file data, as a file, to the file space storage on a file system that is accessible to the Web Gateway JEE application.

8.  The web agent sends a message to the agent queue manager, to inform the Web Gateway JEE application that a file has arrived.

9.  The Web Gateway JEE application receives the notification message sent from the web agent via the agent queue manager.

10. The Web Gateway JEE application updates a database that contains information about the files that are stored in file spaces.

11. **11** The Web Gateway JEE application sends a response, which is destined for the web agent, to the agent queue manager.

12. The web agent receives the response message and completes the file transfer operation.

13. At some later time, a user or process makes a RESTful HTTP request to the Web Gateway JEE application, to retrieve a file from the user's file space. In this diagram the request is made by a web browser. The request can be made by any HTTP client.

14. The Web Gateway JEE application receives the HTTP request, decodes it, and uses the file space information database to locate the file data.

15. The Web Gateway JEE application reads the file data from the file space storage, which is located on a file system that is accessible from the Web Gateway JEE application.

16. The Web Gateway JEE application sends the file data back to the web browser that requested it.

17. The web browser writes the file data to the file system on the destination system.

### Sign-on the Web Gateway

1.  From the prospective STUDENT’s computer (base system), open a browser (Firefox) and point it to the following address:

    <img src="media/image116.png" width="497" height="120" />

    http://192.168.91.129:9080/wmqftesamples/LoginForm.html

    <img src="media/image116.png" width="497" height="120" />

2.  You are presented with the WMQ FTE Example login page.

    <img src="media/image117.png" width="624" height="181" />

3.  Enter the login details:

    Username: STUDENT1

    Password: Pa55word

<!-- -->

1.  <embed src="media/image118.emf" width="408" height="217" />

<!-- -->

1.  Click the “Login” button once you have entered the UserID / Password. The following screen should then be displayed:

    <embed src="media/image119.emf" width="681" height="258" />

### Configure Settings

1.  Click Settings. Here you can set a context root for the Web Gateway. In this case we will use the default **wmqfte**. You can also configure destination mappings here. Destination settings are arbitrary names which are meaningful to the users’ environment. Destination mappings specify a WMQ MFT agent which will send / receive the transfers to / from the Web Gateway.

    <img src="media/image120.png" width="653" height="386" />

2.  Agent addresses are specified with the agent-name@qmgr-name syntax. Change the destination queue manager to **MB8QMGR** for both Accounts and Marketing. As you can see, you are specifying AGENT1@MB8QMGR for the Accounts destination and AGENT2@MB8QMGR for the Marketing destination. MB8QMGR is the agent queue manager for both agents.

    <img src="media/image121.png" width="598" height="198" />

3.  Click **Save** then **OK**.

### Uploading files

> <img src="media/image122.png" width="529" height="242" />
>
> Figure 3.15.3

1.  You can upload a file to the Web Gateway using an HTTP client. The application server that is hosting the Web Gateway application receives the HTTP request and the file is temporarily stored until the web agent starts to transfer it. The web agent transfers the file to the agent that was named as the destination agent in the original transfer request. As shown in Figure 3.15.1, there is no need for the HTTP client that submitted the transfer request to have an agent installed. The destination system must have an agent installed, and the system hosting the Web Gateway application must have a web agent installed.

<!-- -->

1.  Clickthe <img src="media/image123.png" width="68" height="18" /> Tab which will then display the following screen:

    <img src="media/image124.png" width="584" height="317" />

2.  Click Browse and navigate to C:\\Demos\\Test\_files.

    <img src="media/image125.png" width="541" height="353" />

3.  Select file FCOMAS1 and click Open.

    <img src="media/image126.png" width="606" height="278" />

4.  Leave the **Send to** and **Overwrite destination file?** Fields as the defaults.

5.  Click **Upload**.

    <img src="media/image127.png" width="428" height="285" />

6.  You will receive a pop-up window about the Upload Status indicating you sending a file to the Web Gateway. It is being sent over HTTP. Even though there is a MFT agent (STUDENT1) running on the MFT client (student image) machine, that agent is not involved in sending the file to the Web Gateway.

    <img src="media/image128.png" width="212" height="85" />

7.  Click OK to close the pop-up.

8.  You will receive another pop-up indicating that a transfer has been submitted. The file is now on the Web Gateway which is running on the IBM Application Server on the MFT Server (Teacher image). The Web Gateway then submits a transfer request to the final destination.

    <img src="media/image129.png" width="145" height="85" />

|                                                                           |                                                          |
|---------------------------------------------------------------------------|----------------------------------------------------------|
| <img src="media/image38.png" alt="sign-caution" width="57" height="57" /> | Important!                                               
                                                            
  PLEASE DO NOT clickthe X as this will DELETE the FILES..  |

1.  On the MFT Server (Teacher image), check the transfer log. Referring to the Managed File Transfer – Current Transfer Progress on the bottom pane you see that agent WEB\_AGENT1 sent the file FCOMAS1.csv to AGENT1 which is the agent for the destination ACCOUNTS. On the browser, we specified ACCOUNTS as the destination since that is meaningful to a user.

    <img src="media/image130.png" width="662" height="401" />

    Even though both agents are running on the MFT Server (queue manager MB8QMGR) in our POT environment, this would normally not be the case. The Web Gateway agent WEB\_AGENT1 of course needs to run locally to the application server, but AGENT1 could be running on any machine in the MFT network.

### Summary

Congratulations! You used the Web Gateway to extend an existing IBM MQ Managed File Transfer network to support clients that use the HTTP protocol. The Web Gateway provides a link from clients that are using the HTTP protocol into a IBM MQ Managed File Transfer network that already exists. Transfers that use the Web Gateway are logged throughout the transfer.

This concludes Lab 3.

## CONGRATULATIONS! 

### You have completed this hands-on lab.

[Return MQ MFT Menu](mq_mft_pot_overview.html)
