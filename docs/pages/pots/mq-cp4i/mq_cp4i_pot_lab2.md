---
title: Create a Multiinstance Queue Manager Using YAML
toc: true
sidebar: labs_sidebar
folder: pots/mq-cp4i
permalink: /mq_cp4i_pot_lab2.html
summary: Learn How to Code YAML for OpenShift Objects
applies_to: [administrator,developer]
---

# Lab 2 - Creating a Multiinstance Queue Manager Using YAML

Kubernetes uses replicas to keep a queue manager running. But this does not provide a full high availability environment. Multi-instance queue managers can be configured in the Cloud Pak for Integration for HA. 

## Lab objectives

These instructions will document the process to deploy a highly available (HA) persistent IBM MQ on the Cloud Pak for Integration (CP4I).

You will know how to set up MQ in a highly available topology where there is an active and passive container running.

### Pre-reqs

You should have already downloaded the artifacts for this lab in the lab Environment Setup from [GitHub MQonCP4I](https://github.com/ibm-cloudintegration/mqoncp4i). 

If you are doing this lab out of order return to [Environment Setup](mq_cp4i_pot_envsetup.html) to perform the download. Then continue from here.

#### Important points to note

The lab guide assumes you are using the RHEL Virtual Desktop Image (VDI) VM from the IBM Technology Zone. If you are using another platform, you can download the necessary artifacts from the github repo. The instructor will provide directions.

If running as part of a PoT, you will only see your project (namespace). The name will be of the form *clustername* + *your student number*. For instance if the cluster name is **chopper** and your student number is **10**, your namespace will be **chopper10**. So each attendee has a unique namespace and will only be authorized to see that namespace. Within your namespace you will only find your queue manager **mq10mi** in this example. You will also find a previously configured queue manager *qmgrxx*, where xx = your student number. That queue manager will not be used in this PoT and can be ignored.

{% include important.html content="You will see other projects such as cp4i-ace, cp4i-api, cp4i-mq. Since this cluster will be shared with other PoTs those have been predefined. They are not to be used for this PoT and can be ignored. You will only use your assigned namespace and at times *cp4i*. *cp4i-mq* was used to document part of this lab. Where you see *cp4i-mq*, you will substitute your assigned namespace." %}

{% include tip.html content="The screen shots were taken on a test cluster and many will not match what you see when running the lab. Particularly URL values will be different depending on the cluster where CP4I is running. Projects (Namespaces) may also vary. It is important to follow the directions, not the pictures." %}

#### Further information

[IBM MQ Knowledge Center](https://www.ibm.com/docs/en/cloud-paks/cp-integration/2021.1?topic=guide-creating-new-queue-manager-red-hat-openshift)  
## Deploy the MQ Queue Manager with associated resources

### Define the queue manager instance of CP4I in a yaml file

To create an MQ queue manager, you could use the UI as you did in Lab 1 to create each queue manager needed. Most administrators use scripting to define things which novice users would do with the UI.

This lab shows you how to write yaml to create the multi-instance queue manager.

1. You should still be logged into the OpenShift environment. If not, click on your username on the top right menu of the OpenShift Console, then click on *Copy Login Command*. 

	![](./images/pots/mq-cp4i/lab2/image00.png)

1. Click *Display Token*, copy the token and run it on your terminal.
	
	![](./images/pots/mq-cp4i/lab2/image0a.png)
	
	{% include note.html content="You should still be in your personal project so you shouldn't need to run this command unless your login has timed out." %}
	
	![](./images/pots/mq-cp4i/lab2/image0c.png)	
	 If not in your assigned project, run the following command to navigate to your assigned project substituting your personal namespace for palpatine15:
	
	```
	oc project palpatine15
	``` 
	
	![](./images/pots/mq-cp4i/lab2/image0d.png)

1. Open a new terminal window by double-clicking the icon on the desktop.

	![](./images/pots/mq-cp4i/lab2/image1a.png)

1. Navigate to the *MQonCP4I/multiinstance/deploy* directory using the following commnand:

	```
	cd MQonCP4I/multiinstance/deploy
	```
	
1. Enter the following command to display the permissions for the files:

	```
	ls -al
	```
	
	![](./images/pots/mq-cp4i/lab2/image201.png)
	
1. Make *installl.sh* and *cleanup.sh* executable with the following commands.

	```
	chmod +x install.sh
	```
	
	```
	chmod +x cleanup.sh
	```

	![](./images/pots/mq-cp4i/lab2/image202.png)
	
1. Enter the following command to edit the file *install.sh*.

	```
	gedit install.sh &
	```
	
	![](./images/pots/mq-cp4i/lab2/image203.png)
	
	{% include note.html content="Adding *&* at the end of the above command will run gedit in the background freeing up your terminal." %}
	
1.	Change the value for *TARGET_NAMESPACE* to your project name. Click the hamburger menu in top right corner and select *Find and Replace*.
	
	![](./images/pots/mq-cp4i/lab2/image204.png)		
1. Enter *00* in the "Find" field and *xx* in the "Replace with" field where *xx* is your student number. Use your student number in place of 00. Click *Replace All*, then click *Save*.

	![](./images/pots/mq-cp4i/lab2/image205.png)
	
	{% include note.html content="The storage class on the COC clusters is **managed-nfs-storage**. If you are an IBMer running on on ROKS then you also need to change *SC* to **ibmc-file-gold-gid**." %}
	
	![](./images/pots/mq-cp4i/lab2/image205b.png)
	
1. In the editor click the drop-down in the *Open* box, click *Other Documents, then select **cleanup.sh**.

	![](./images/pots/mq-cp4i/lab2/image205a.png)
	
	![](./images/pots/mq-cp4i/lab2/image206.png)

1. As above you did above, change *TARGET_NAMESPACE* to your project name, then change "00" in the export commands to your student ID. Click *Save*.

	![](./images/pots/mq-cp4i/lab2/image207.png)
	
	![](./images/pots/mq-cp4i/lab2/image207a.png)
	
1. As done previously click drop-down next to *Open*, click *Other Documents*, then select *mqmultiinstance.yaml_template*.

	![](./images/pots/mq-cp4i/lab2/image208.png)
	
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
	
	![](./images/pots/mq-cp4i/lab2/image209.png)
	
	The results are displayed showing the objects (mentioned above) which were created.
		
1. Open your web browser tab for the Platform Navigator. Refresh the page. The status for *mqxx-multiinstancemq* will have a status of *Pending* until completely deployed. 

	To see the runtimes only, click the hamburger menu in top-left corner, click drop-down for Administration, then select *Integration runtimes*. 
	
	![](./images/pots/mq-cp4i/lab2/image211a.png) 
	
1. After a couple of minutes the instance is deployed and the status changes to *Ready*. 

	![](./images/pots/mq-cp4i/lab2/image211b.png)

	{% include note.html content="If you have closed the Platform Navigator, you can change to the OCP Console > Project cp4i > Networking > Routes > cp4i-navigator-pn. See screen shots below." %}
	
	![](./images/pots/mq-cp4i/lab2/image211c.png)
		
### Access and display your MQ instance in the Platform Navigator

1. Click the hyperlink for your instance which will take you to the MQ Console for your queue manager, but first you must respond to the security warnings if prompted. Click *Advanced* then *Accept thee Risk and Continue*.

1. You are then taken to the MQ Console for your queue manager. Recall that you named the queue manager with your student ID prefix and *mi* - **mq00mi**. The instance name, **mq00-multiinstancemq** has the same prefix but we shortened it for the queue manager name. Click *Manage* then *Local queue managers*.

	![](./images/pots/mq-cp4i/lab2/image213a.png)
	
	{% include tip.html content="You could easily just click *Manage mqxxmi* to bypass the *Local queue managers* step." %}
	
	![](./images/pots/mq-cp4i/lab2/image213.png)
	
1. On the *Manage* page you will see *Queues*, *Topics*, *Subscriptions*, and *Communications*. Under *Queues* you will see **APPQ** which was defined by the mqsc commands in the yaml defined *ConfigMap*. 

	![](./images/pots/mq-cp4i/lab2/image214.png)
	
1. Click *Communication* > *App Channels* where you will find the channel **MQxxCHL**. The channel was also defined in the mqsc commands in the yaml defined *ConfigMap*. Click the elipsis on the right side then select *View Configuration* to display or edit the channel properties. 

	![](./images/pots/mq-cp4i/lab2/image215.png)
	
	Click the breadcrumb to return to *Manage* page.
	
	![](./images/pots/mq-cp4i/lab2/image216.png)
	
1. Notice the *View Configuration* hyperlink in the top right corner. This takes you to the queue manager properties. Click the link now.
	
	![](./images/pots/mq-cp4i/lab2/image212.png)
	
1. You arrive at the overall queue manager properties to be displayed or edited. Click *Security* > *Channel authentication*. Here you see the channel auth created by the mqsc commands in the yaml defined *ConfigMap* as well as the system channel auth records. Your yaml file defined the MQxxCHL record with a type of *Block*. You can click the wrench icon to display or edit the record. If you do change the properties you will then need to refresh security. To do that you simply click the *Actions* elipsis and select the necessary refresh option.

	![](./images/pots/mq-cp4i/lab2/image217.png)
	
1. Continue to explore the MQ Console as long as you like.

### Review your MQ instance in OpenShift

There is an extensive amount of detail in the OpenShift Console. The following  instructions will lead you to accomplish the purpose of this lab. But feel free to explore more detail to help you understand OpenShift and CP4I. 
	
1. Return to the browser tab in Firefox for the OCP Console. You will be in the *Administrator* view. Click *Projects* then scroll down to find your namespace and click the hyperlink. 

	![](./images/pots/mq-cp4i/lab2/image218.png)
	
1. The project *Overview* opens where you can see the status and utilization of your namespace (project).

	![](./images/pots/mq-cp4i/lab2/image219.png)
	
1. On the left sidebar expand *Operators*, then select *Installed Operators*.
	
	![](./images/pots/mq-cp4i/lab2/image220.png)
	
1. Scroll to find **IBM MQ** then click the hyperlink. 
	
	![](./images/pots/mq-cp4i/lab2/image221.png)

	You can read about the *IBM MQ* operator and select the tabs to discover what information is available. Click the *Queue Manager* tab. You will see that your instance is running. 

	![](./images/pots/mq-cp4i/lab2/image222.png) 
	
1. On the left sidebar expand *Workloads*, then select *Pods*. You will see two pods for your MQ instance. Each pod has a replica count of one, meaning there is one container in the pod and it is running your queue manager. Pod *mqxx-multiinstancemq-ibm-mq-0* shows 1/1 while the other pod *mq00-multiinstancemq-ibm-mq-1* shows 0/1, meaning there is one container in the pod but it is not ready (in standby mode). 

	![](./images/pots/mq-cp4i/lab2/image223.png)
	
1. You can also see this on the command line. In your terminal window enter the command using your student ID in place of *xx*.

	```
	oc get pods | grep mqxx
	```
	
	![](./images/pots/mq-cp4i/lab2/image224.png)

1. In the OCP console, click the running pod *mqxx-multiinstancemq-ibm-mq-0*. The *Pod Details* page opens showing resource utilitzation.

	![](./images/pots/mq-cp4i/lab2/image225.png)
	
1. Scroll down to the *Volumes* section. Here you find *Persistent Volume Claims* (PVCs) for the pod. Notice the *Mount Path*. If you are familiar with multi-instance queue managers, you know that *mqxx-multiinstancemq-ibm-mq-persisted-data* and *mqxx-multiinstancemq-ibm-mq-recovery-logs* are the shared data between active and standby queue managers.

	![](./images/pots/mq-cp4i/lab2/image226.png)
	
	Verify this by repeating the above for pod *mqxx-multiinstancemq-ibm-mq-1*.
	
## Test the deployment

1. In a terminal window navigate to */home/ibmuser/MQonCP4I/multiinstance/test* directory. You will find three files (and additional files for TLS): 

	* CCDT.JSON
	* getMessage.sh
	* sendMessage.sh

	Open them in gedit. 
	
	![](./images/pots/mq-cp4i/lab2/image227.png)

1. In the *ccdt.json* file, you need to update the host next to *host:* with your host name. To get your host name, run the following command in a terminal window substituting your student number for xx:

	```
	oc get route | grep mqxx
	```
	
	Your host name should start with *mqxx-multiinstancemq-ibm-mq-qm* where xx is your student ID.
	
	![](./images/pots/mq-cp4i/lab2/image228.png)
	
1.	It is much easier to read in the OpenShift console. Return to the web browser tab where the console is open. On the left sidebar, scroll down to *Networking*, use the drop-down to see the options, and select *Routes*. 

	Use the filter to search for your queue manager routes to limit your search much like you did using *grep* with the oc command. Click the route name hyperlink for *mqxx-multiinstancemq-ibm-mq-qm*.
	
	![](./images/pots/mq-cp4i/lab2/image229.png)
	 
1. Under *Route Details* scroll down to *Router:default*. There you will find the *Host* URL. Copy the URL under *Host* to use in the ccdt.json file.
	 
	 ![](./images/pots/mq-cp4i/lab2/image230.png)
	 
1. Return to the editor and paste the value into the *host:* field of ccdt.json. Under *channel* > *name:* insert the name of your SVRCONN channel (MQxxCHL). Then input the value of your queue manager in the *queueManager:* field.

	![](./images/pots/mq-cp4i/lab2/image231.png)
	
	Click *Save* to save ccdt.json.
	
1. Now edit *getMessage.sh* and *sendMessge.sh*. You need to change the same values in each file. In the export statements, change *00* to your student ID. Change the paths for *MQCCDTURL* and *MQSSLKEYR* to **/home/student/...**. Click the *Save* button for each file. 

	![](./images/pots/mq-cp4i/lab2/image232a.png)
	
	![](./images/pots/mq-cp4i/lab2/image232b.png)	
1. In the terminal window in the *./test* directory make *sendMessage.sh* and *getMessage.sh* files executable with the following commnads:

	```
	chmod +x sendMessage.sh
	```
	
	```
	chmod +x getMessage.sh
	```
	
	![](./images/pots/mq-cp4i/lab2/image234.png)	
1. In the terminal window in the *./test* directory initiate the testing by running the following command: 

	```
	./sendMessage.sh
	```
	
	The script will then connect to MQ and start sending messages incessantly. Leave this window open to keep sending messages.

1. Open another command window, navigate to */home/student/MQonCP4I/multiinstance/test* directory and run the following command: 

	```
	./getMessage.sh
	```
	
	You should get a list of the all messages that have been previously sent before running the command and the ones that are being sent after. Leave this window open to keep getting the messages.
	
	![](./images/pots/mq-cp4i/lab2/image235.png)

1. Return to the OpenShift Console, expand *Workloads* and select *Pods*. If necessary you can enter your prefix in the filter field so you only see your pods.

	![](./images/pots/mq-cp4i/lab2/image236.png) 

1. To see how the pods work together in action, delete the active pod by clicking the elipsis next to the pod with 1/1 and click *Delete Pod*. Respond *Delete* in the confirmation pop-up. 

	![](./images/pots/mq-cp4i/lab2/image237.png)
	
1. Notice that the pod which was active is now *terminating* and will restart because you asked OpenShift to keep at least one replica.

	![](./images/pots/mq-cp4i/lab2/image240.png)
	
1. Once the active pod is deleted, the connection will then reconnect to the other pod for it to take over. Verify this by observing the active pod in the console.

	![](./images/pots/mq-cp4i/lab2/image238.png)
	
1. Also observe the behavior of the running appplications in the terminals. They have reconnected to the other pod and continue to run. Notice the time to reconnect.

	![](./images/pots/mq-cp4i/lab2/image239.png)
		
1. Try deleting the current active pod and observe again that the standby pod takes over and the applications reconnect and continue. 

	![](./images/pots/mq-cp4i/lab2/image240.png)
	
	![](./images/pots/mq-cp4i/lab2/image241.png)
	
	And observe the applications reconnecting again.
	
	![](./images/pots/mq-cp4i/lab2/image243.png)
	
	![](./images/pots/mq-cp4i/lab2/image244.png)

1. You can now stop the programs by entering *Ctrl-c* in each terminal window. 

## Connecting MQ Explorer to a deployed Queue Manager in OpenShift

These steps document how you can connect MQ Explorer to a Queue Manager running in OpenShift.

1. Open a new terminal window.
	
1. Enter the following command to start MQ Explorer making sure to use the correct case:

	```
	MQExplorer
	```
	
	![](./images/pots/mq-cp4i/lab2/image247.png)	
1. When the utility is ready, right-click *Queue Managers* and select *Add Remote Queue Manager*.

	![](./images/pots/mq-cp4i/lab2/image248.png)

1. Enter your queue manager name using your student ID. Click *Next*.

	![](./images/pots/mq-cp4i/lab2/image249.png)

1. Recall the values from the ccdt.json file you previously updated. 
	Enter the value from the *host:* field in the *Host name of IP address*. This was the URL of the *Router Canonical Host* from the route. 
	Enter **443** for the *Port number*.
	Enter your SVRCONN channel name in the *Server-connection channel* field.
	Click the checkbox for Multi-instance queue manager.
	Enter the same details for the second instance.
	Click *Next* three times.
	
	![](./images/pots/mq-cp4i/lab2/image138.png)
	
	[For more information refer to KnowledgeCenter](https://www.ibm.com/support/knowledgecenter/SSFKSJ_9.1.0/com.ibm.mq.ctr.doc/cc_conn_qm_openshift.htm)
		
1. Click the checkbox for *Enable SSL key repositories*. Click *Browse* and navigate to */home/ibmuser/MQonCP4I/tls* and select **MQExplorer.jks**. Then click *Open*.

	![](./images/pots/mq-cp4i/lab2/image139.png)
	
1. Click the *Enter password* button and enter **password**. Click *OK* then *Next*.

	![](./images/pots/mq-cp4i/lab2/image140.png)
	
1. On the next screen click the checkbox for *Enable SSL options*. Click the drop-down next to *SSL CipherSpec* and select **ANY_TLS12_OR_HIGHER**.

	![](./images/pots/mq-cp4i/lab2/image141.png)
	
1. Click *Finish*. You will get a pop-up saying "Trying to connect to the queue manager".

	![](./images/pots/mq-cp4i/lab2/image141a.png)
	
1. After a few seconds you see that MQ Explorer has connected. The Queue Manager will be added and shown in the navigator.

	![](./images/pots/mq-cp4i/lab2/image142.png)
	
1. Operate MQ Explorer as you normally would. Expand the queue manager and "explore" MQ looking at queues, channels, etc.

	![](./images/pots/mq-cp4i/lab2/image143.png)

1. When done "exploring" you may close MQ Explorer.

## Cleanup

If you are doing this lab during the MQ on CP4I POT you should complete the cleanup to conserve resources on the cluster.

If you are an IBMer running the lab on your ROKS cluster it is up to you whether you want to leave this queue manager for demo purposes. If not then you should complete the cleanup. 

1. Close all the applications and terminal windows.

1. In a new terminal, navigate to /home/student/MQonCP4I/streamq/deploy:

	```sh
	cd ~/MQonCP4I/multiinstance/deploy
	```
	
	You should have updated the cleanup.sh script earlier in the lab. Run it now to delete the multiinstance queue manager.
	
	```sh
	./cleanup.sh
	```

	![](./images/pots/mq-cp4i/lab2/image245.png)  
  
  
[Continue to Lab 3](mq_cp4i_pot_lab3.html)

[Return MQ CP4I Menu](mq_cp4i_pot_overview.html) 