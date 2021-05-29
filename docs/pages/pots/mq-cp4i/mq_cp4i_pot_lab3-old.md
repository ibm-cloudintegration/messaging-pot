---
title: Create a Multiinstance Queue Manager Using YAML
toc: false
sidebar: labs_sidebar
folder: pots/mq-cp4i
permalink: /mq_cp4i_pot_lab3.html
summary: Learn How to Code YAML for OpenShift Objects
applies_to: [administrator,developer]
---

# Lab 3 - Creating a Multiinstance Queue Manager Using YAML

Kubernetes uses replicas to keep a queue manager running. But this does not provide a full high availability environment. Multi-instance queue managers can be configured in the Cloud Pak for Integration for HA. 

## Lab objectives

These instructions will document the process to deploy a highly available (HA) persistent IBM MQ on the Cloud Pak for Integration (CP4I).

You will know how to set up MQ in a highly available topology where there is an active and passive container running.


## Deploy the MQ Queue Manager with associated resources

### Define the queue manager instance of CP4I in a yaml file

To create a MQ cluster, you could use the UI as you did in Lab 2 to create each queue manager in your cluster. Most administrators use scripting to define things which novice users would do with the UI.

This lab shows you how to write yaml to create the multi-instance queue manager.

1. You should still be logged into the OpenShift environment. If not, click on your username on the top right menu of the OpenShift Console, then click on *Copy Login Command*. 

	![](./images/pots/mq-cp4i/lab3/image00.png)

1. Click *Display Token*, copy the token and run it on your terminal.
	
	![](./images/pots/mq-cp4i/lab3/image0a.png)
	
	{% include note.html content="You should still be in project *cp4i-mq* so you shouldn't need to run this command." %}
	
	Run the following command to navigate to the *mq* project:
	
	```
	oc project cp4i-mq
	``` 

1. Open a new terminal window by double-clicking the icon on the desktop.

	![](./images/pots/mq-cp4i/lab3/image1a.png)

1. Navigate to the *MQonCP4I/multiinstance/deploy* directory using the following commnand:

	```
	cd MQonCP4I/multiinstance/deploy
	```

	![](./images/pots/mq-cp4i/lab3/image101.png)

1. Enter the following command to edit the file *install.sh*.

	```
	gedit install.sh
	```

	![](./images/pots/mq-cp4i/lab3/image102.png)

1. Change "00" in the export commands to your student ID. Click *Save*.

	![](./images/pots/mq-cp4i/lab3/image103.png)

1. In the editor click the drop-down in the *Open* box, type "c", then select **cleanup.sh**.

	![](./images/pots/mq-cp4i/lab3/image104.png)

1. Change "00" in the export commands to your student ID. Click *Save*.

	![](./images/pots/mq-cp4i/lab3/image105.png)
	
1. As done previously click drop-down next to *Open*, click *Other Documents*, then select *mqmultiinstance.yaml_template*.

	![](./images/pots/mq-cp4i/lab3/image106.png)
	
1. Do not change anything in this file! The *install.sh* script will copy this file to *mqmultiinstance.yaml*, substitute your student ID in the environment varibles, and apply the yaml file to the OpenShift cluster. Review the file to understand how your queue manager will be created. 

	Pay particular attention to each *kind* stanza
	* 	**ConfigMap** for *mqsc* commands
	*  **Secret** to define the *tls* certificate and key
	*  **QueueManager** to define attributes for your specific QM
	*  **Route** to accesss the queue manager from outside the cluster

	In the *QueueManager* stanza, notice the name, the type (MultiInstance) and references to the *ConfigMap* and *Secret*.
	
1. Open a new terminal window and navigate to the deploy directory again with the command: 

	```
	cd MQonCP4I/multiinstance/deploy
	```

1. Run the *install.sh* script:

	```
	./install.sh
	```
	
	![](./images/pots/mq-cp4i/lab3/image107.png)
	
	The results are displayed showing the objects (mentioned above) which were created.
		
1. Open a web browser by double-clicking the *Firefox* icon on the desktop. 

	![](./images/pots/mq-cp4i/lab3/image8.png)
	
1. Navigate to the *Platform Navigator* by clicking the bookmark. 

	![](./images/pots/mq-cp4i/lab3/image109.png)

1. After a couple of minutes the instance is deployed and the status changes to *Ready*. 

	![](./images/pots/mq-cp4i/lab3/image110.png)
	
	You may click *Show less* to conserve screen space.
	
### Access and display your MQ instance in the Platform Navigator

1. Click the hyperlink for your instance which will take you to the MQ Console for your queue manager, but first you must respond to the security warning. Click *Advanced* then *Accept thee Risk and Continue*.

	![](./images/pots/mq-cp4i/lab3/image11.png)

1. You are then taken to the MQ Console for your queue manager. Recall that you named the queue manager with your student ID prefix and *mi* - **mq00mi**. The instance name, **mq00mi**, is the same as the queue manager name. 

	![](./images/pots/mq-cp4i/lab3/image111.png)

1. Click *Manage*, you will see *Queues*, *Topics*, *Subscriptions*, and *Communications*. Under *Queues* you will see **APPQ** which was defined by the mqsc commands in the yaml defined *ConfigMap*. 

	![](./images/pots/mq-cp4i/lab3/image112.png)
	
1. Click *Communication* > *App Channels** where you will find the channel **MQ00CHL**. The channel was also defined in the mqsc commands in the yaml defined *ConfigMap*. Click the elipsis on the right side then select *Configuration* to display or edit the channel properties. 

	![](./images/pots/mq-cp4i/lab3/image113.png)
	
	Click the breadcrumb to return to *Manage* page.
	
	![](./images/pots/mq-cp4i/lab3/image114.png)
	
	Notice the *Configuration* hyperlink in the top right corner. This takes you to the queue manager properties. Click the link now.
	
	![](./images/pots/mq-cp4i/lab3/image114a.png)
	
1. You arrive at the overall queue manager properties to be displayed or edited. Click *Security* > *Channel authentication*. Here you see the channel auth created by the mqsc commands in the yaml defined *ConfigMap* as well as the system channel auth records. Your yaml file defined the MQ00CHL record with a type of *Block*. You can click the wrench icon to display or edit the record. If you do change the properties you will then need to refresh security. To do that you simply click the *Actions* elipsis and select the necessary refresh option.

	![](./images/pots/mq-cp4i/lab3/image15.png)
	
1. Continue to explore the MQ Console as long as you like.

### Review your MQ instance in OpenShift

There is an extensive amount of detail in the OpenShift Console. The following  instructions will lead you to accomplish the purpose of this lab. But feel free to explore more detail to help you understand OpenShift and CP4I. 

If running as part of a PoT there will be multiple mq instances and queue managers running so you need to be looking for your *mqxxmi* instance queue manager.
 	
1. Open a new browser tab in Firefox and click the *Kylo - OCP* bookmark. Remember to click the appropriate bookmark for the clust you are running in.

	![](./images/pots/mq-cp4i/lab3/image116.png)
	
1. You will be in the *Administrator* view. Click *Projects* then scroll down to find **cp4i-mq** and click the hyperlink.

	![](./images/pots/mq-cp4i/lab3/image117.png)
	
1. The project *Overview* opens where you can see the status and utilization of the **cp4i-mq** namespace (project).

	![](./images/pots/mq-cp4i/lab3/image118.png)
	
1. On the left sidebar expand *Operators*, then select *Installed Operators*.
	
	![](./images/pots/mq-cp4i/lab3/image119.png)
	
1. Scroll to find **IBM MQ** then click the hyperlink. 
	
	![](./images/pots/mq-cp4i/lab3/image120.png)

	You can read about the *IBM MQ* operator and select the tabs to discover what information is available. Click the *Queue Manager* tab. You will see that your instance is running. 

	![](./images/pots/mq-cp4i/lab3/image121.png) 
	
1. On the left sidebar expand *Workloads*, then select *Pods*. You will see two pods for your MQ instance. Each pod has a replica count of one, meaning there is one container in the pod and it is running your queue manager. Pod *mq00mi-ibm-mq-0* shows 1/1 while the other pod *mq00mi-ibm-mq-1* shows 0/1, meaning there is one container in the pod but it is not ready (in standby mode). 

	![](./images/pots/mq-cp4i/lab3/image122.png)
	
1. You can also see this on the command line. In your terminal window enter the command using your student ID in place of *00*.

	```
	oc get pods | grep mq00
	```
	
	![](./images/pots/mq-cp4i/lab3/image123.png)

1. In the OCP console, click the running pod *mq00mi-ibm-mq-0*. The *Pod Details* page opens showing resource utilitzation.

	![](./images/pots/mq-cp4i/lab3/image124.png)
	
1. Scroll down to the *Volumes* section. Here you find *Persistent Volume Claims* (PVCs) for the pod. Notice the *Mount Path*. If you are familiar with multi-instance queue managers, you know that *mq00mi-ibm-mq-persisted-data* and *mq00mi-ibm-mq-recovery-logs* are the shared data between active and standby queue managers.

	![](./images/pots/mq-cp4i/lab3/image125.png)

## Test the deployment

1. In a terminal window navigate to */home/ibmuser/MQonCP4I/multiinstance/test* directory. You will find three files (and additional files for TLS): 

	* CCDT.JSON
	* getMessage.sh
	* sendMessage.sh

	Open them in gedit. 
	
	![](./images/pots/mq-cp4i/lab3/image126.png)

1. In the *ccdt.json* file, you need to update the host next to *host:* with your host name. To get your host name, run the following command in a terminal window:

	```
	oc get route | grep mq00
	```
	
	Your host name should start with *mqxxmi-ibm-mq-qm* where xx is your student ID.
	
	![](./images/pots/mq-cp4i/lab3/image127.png)
	
1.	It is much easier to read in the OpenShift console. Return to the web browser tab where the console is open. On the left sidebar, scroll down to *Networking*, use the drop-down to see the options, and select *Routes*. 

	Use the filter to search for your queue manager routes to limit your search much like you did using *grep* with the oc command.
	
	 ![](./images/pots/mq-cp4i/lab3/image128.png)
	
	 Click the route name hyperlink for *mqxxmi-ibm-mq-qm*.
 
1. Under *Route Details* scroll down to *Router:default*. There you will find the *Host* URL. Copy the URL under *Host* to use in the ccdt.json file.
	 
	 ![](./images/pots/mq-cp4i/lab3/image129.png)
	 
1. Return to the editor and paste the value into the *host:* field of ccdt.json. Under *channel* > *name:* insert the name of your SVRCONN channel (MQxxCHL). Then input the value of your queue manager in the *queueManager:* field.

	![](./images/pots/mq-cp4i/lab3/image130.png)
	
	Click *Save* to save ccdt.json.
	
1. Now edit *getMessage.sh* and *sendMessge.sh*. You need to change the same values in each file. In the export statements, change *00* to your student ID. Click the *Save* button for each file. 

	![](./images/pots/mq-cp4i/lab3/image131.png)

1. In the terminal window in the *./test* directory initiate the testing by running the following command: 

	```
	./sendMessage.sh
	```
	
	The script will then connect to MQ and start sending messages incessantly. Leave this window open to keep sending messages.

1. Open another command window, navigate to */home/ibmuser/MQonCP4I/multiinstance/test* directory and run the following command: 

	```
	./getMessage.sh
	```
	
	You should get a list of the all messages that have been previously sent before running the command and the ones that are being sent after. Leave this window open to keep getting the messages.
	
	![](./images/pots/mq-cp4i/lab3/image132.png)

1. Return to the OpenShift Console, expand *Workloads* and select *Pods*. If necessary you can enter your prefix in the filter field so you only see your pods.

	![](./images/pots/mq-cp4i/lab3/image32.png) 

1. To see how the pods work together in action, delete the active pod by clicking the elipsis next to the pod with 1/1 and click *Delete Pod*. Respond *Delete* in the confirmation pop-up. 

	![](./images/pots/mq-cp4i/lab3/image33.png)

1. Once the active pod is deleted, the connection will then reconnect to the other pod for it to take over. Verify this by observing the active pod in the console.

	![](./images/pots/mq-cp4i/lab3/image33a.png)
	
1. Also observe the behavior of the running appplications in the terminals. They have reconnected to the other pod and continue to run.

	![](./images/pots/mq-cp4i/lab3/image133.png)
		
1. Try deleting the current active pod and observe again that the standby pod takes over and the applications reconnect and continue. 

	![](./images/pots/mq-cp4i/lab3/image134.png)
	
	![](./images/pots/mq-cp4i/lab3/image135.png)
	
	And observer the applications reconneting again.
	
	![](./images/pots/mq-cp4i/lab3/image136.png)

1. You can now stop the programs by entering *ctrl-c* in each terminal window. 

## Connecting MQ Explorer to a deployed Queue Manager in OpenShift

These steps document how you can connect MQ Explorer to a Queue Manager running in OpenShift.


1. Open a new terminal window.
	
1. Enter the following command to start MQ Explorer making sure to use the correct case:

	```
	MQExplorer
	```
	
	![](./images/pots/mq-cp4i/lab3/image36.png)	
1. When the utility is ready, right-click *Queue Managers* and select *Add Remote Queue Manager*.

	![](./images/pots/mq-cp4i/lab3/image37.png)

1. Enter your queue manager name using your student ID. Click *Next*.

	![](./images/pots/mq-cp4i/lab3/image137.png)

1. Recall the values from the ccdt.json file you previously updated. 
	Enter the value from the *host:* field in the *Host name of IP address*. This was the URL of the *Router Canonical Host* from the route. 
	Enter **443** for the *Port number*.
	Enter your SVRCONN channel name in the *Server-connection channel* field.
	Click the checkbox for Multi-instance queue manager.
	Enter the same details for the second instance.
	Click *Next* three times.
	
	![](./images/pots/mq-cp4i/lab3/image138.png)
	
	[For more information refer to KnowledgeCenter](https://www.ibm.com/support/knowledgecenter/SSFKSJ_9.1.0/com.ibm.mq.ctr.doc/cc_conn_qm_openshift.htm)
		
1. Click the checkbox for *Enable SSL key repositories*. Click *Browse* and navigate to */home/ibmuser/MQonCP4I/tls* and select **MQExplorer.jks**. Then click *Open*.

	![](./images/pots/mq-cp4i/lab3/image139.png)
	
1. Click the *Enter password* button and enter **password**. Click *OK* then *Next*.

	![](./images/pots/mq-cp4i/lab3/image140.png)
	
1. On the next screen click the checkbox for *Enable SSL options*. Click the drop-down next to *SSL CipherSpec* and select **ECDHE_RSA_AES_128_CBC_SHA256**.

	![](./images/pots/mq-cp4i/lab3/image141.png)
	
1. Click *Finish*. You will get a pop-up saying "Trying to connect to the queue manager".

	![](./images/pots/mq-cp4i/lab3/image43.png)
	
1. After a few seconds you see that MQ Explorer has connected. The Queue Manager will be added and shown in the navigator.

	![](./images/pots/mq-cp4i/lab3/image142.png)
	
1. Operate MQ Explorer as you normally would. Expand the queue manager and "explore" MQ looking at queues, channels, etc.

	![](./images/pots/mq-cp4i/lab3/image143.png)

1. When done "exploring" you may close MQ Explorer.

## Cleanup

1. Return to the Platform Navigator. Under *Runtimes*, find your instance, click the elipsis on the right and select **Delete**. 

	![](./images/pots/mq-cp4i/lab3/image46.png)
	
1. Enter the instance name to confirm the deletion.

	![](./images/pots/mq-cp4i/lab3/image145.png)
	
	This will delete the queue manager pods and all related artifacts. This will help reduce load on the cluster as you continue the rest of the labs. This queue manager will not be needed again. 
	
1. You also need to cleanup *PVCs* which do not get deleted automatically. In one of the terminal windows navigate to */home/ibmuser/MQonCP4I/multiinstance/deploy* directory. Enter the following command:

	```
	./cleanup.sh
	```

	![](./images/pots/mq-cp4i/lab3/image144.png)
   
[Continue to Lab 4](mq_cp4i_pot_lab4.html)

[Return MQ CP4I Menu](mq_cp4i_pot_overview.html) 