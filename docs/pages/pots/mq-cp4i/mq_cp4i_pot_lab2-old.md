---
title: Creating an MQ Instance Using the Platform Navigator
toc: false
sidebar: labs_sidebar
folder: pots/mq-cp4i
permalink: /mq_cp4i_pot_lab2.html
summary: Create an MQ Queue Manager from Navigator
applies_to: [administrator,developer]
---

# Lab 2 - Creating an MQ Instance Using the Platform Navigator

Starting with this lab, each attendee will be assigned an ID number (01 - 30) by the instructor if running a PoT. If running this lab independently, use ID number 00. During a PoT the instructor will always be ID 00.

These instructions assume you are using the Skytap desktop VM for the *sMQ on CP4I PoT*.

## Getting Started with MQ on Cloud Pak for Integration 

These instructions document how to setup MQ within Cloud Pak for Integration which is accessible from within the OpenShift Cluster. The instructions have been created using a fresh OpenShift environment deployed on IBM Cloud RedHat OpenShift Kubernetes Service (ROKS), however the process should be similar on other environments.

## Deploying IBM MQ for internal consumers

1. Open a new Firefox web browser tab and click the *Kylo* bookmark.

	![](./images/pots/mq-cp4i/lab2/image101.png)
	
	{% include note.html content="The instructions and screenshots in this lab guide were recorded using the CoC **Kylo** cluster. PoT events will use the CoC Demo Clusters *Biggs*, *Dooku*, or *Obi-Wan*. The instructor will inform you as which cluster you will be using. Substitute that cluster name for *Kylo*." %}
	
1. Click *Show less* hyperlink to save screen space, then click *Runtimes*.

	![](./images/pots/mq-cp4i/lab2/image2.png)
	
1. Click *Create instance* to display the various runtimes available in CP4I.

	![](./images/pots/mq-cp4i/lab2/image3.png)
	
1. A number of tiles are displayed. Click the *Queue Manager* tile, then click *Next*.

	![](./images/pots/mq-cp4i/lab2/image4.png)
	
1. You will use the option called **Quick start** which will deploy an MQ container with 1 cpu, 1 GB of memory, and measured at 0.25 vpc (Virtual Processor Core). Click the *Quick start* tile then click *Next*.

	![](./images/pots/mq-cp4i/lab2/image102.png)
	
1. On the *Create queue manager* configuration page, enter "mq" plus your student number as a prefix to the Name *quickstart-cp4i* and use the drop-down under *Namespace* to select **cp4i-mq**. Click the License acceptance button to turn on.

	![](./images/pots/mq-cp4i/lab2/image103.png)
	
1. Scroll down to *Queue Manager* section. Using the drop-down under *Type of availability* select **SingleInstance**. Under *Type of volume* leave the default **ephemeral**. You have a choice between ephemeral or persistent-claim. Ephemeral means that the queue manager does not maintain state so does not need persistent-storage. 

	![](./images/pots/mq-cp4i/lab2/image104.png)
	
	1. Under *Tracing* click the button to enable tracing and use the drop-down under *Tracing namespace* to select **cp4i-tracing**.

	Do NOT click Create yet. You need to name your queue manager.

	![](./images/pots/mq-cp4i/lab2/image105.png)
	
1.	On the left side bar, click the button under *Advanced settings* to expose more settings. You may need to scroll to find the *Queue Manager* section again. Under *Advanced: Name* field you see the default **QUICKSTART**. Replace this with your queue manager name using your student ID and qs, ie **mq00qs**. 

	Now click *Create*.
	
	![](./images/pots/mq-cp4i/lab2/image106.png)

1. If all entries are valid, you will receive a notification of success in green. Any errors result in a notification in red. Status remains *Pending* while the queue manager is being provisioned. If you click *Pending* you may receive a *Conditions* pop-up. Since you specified *Tracing* in the configuration, you need to register the queue manager to the operations dashboard. Close the pop-up.

	![](./images/pots/mq-cp4i/lab2/image8a.png)

### Register mq queue manager with Operations Dashboard

This only needs to be done once by the first queue manager that is starting. If the *Pending* status changes to *Ready* then registration is complete and you can skip ahead to the *Start MQ Console* section.

[Go to Start MQ Console](#mqconsole)

1.	If the status has changed to *Error* click the *Error* hyperlink. 
	
	![](./images/pots/mq-cp4i/lab2/image9.png)
	
1.	This error occurs because *MQ* has not been registered in the Operations Dashboard in the *tracing* namespace. Close the *Conditions* pop-up then click *Capabilities* on the Platform Navigator.	
	
	![](./images/pots/mq-cp4i/lab2/image10.png)
	
1.	Click the *Operations Dashboard* hyperlink to open the dashboard. 
	
1.	![](./images/pots/mq-cp4i/lab2/image11.png)
	
1.	On the *What's new* window, click the "Don't show again" checkbox then close the window. 
	
	![](./images/pots/mq-cp4i/lab2/image12.png)
	
1.	Once the *Operations Dashboard* is open, click the drop-down for *Manage* on the left sidebar. Then click *Registration requests*.

	![](./images/pots/mq-cp4i/lab2/image13.png)
	
1. The request is displayed with a pod name. This may or may not be a pod for your queue manager (notice the pod name prefix) depending on whether your queue was the first to be deployed. The status is new and needs to approved. Click the *Approve* hyperlink. 

	![](./images/pots/mq-cp4i/lab2/image14.png)
	
1. Click the copy button within the pop-up. This text is a command to create the missing secret for the tracing namespace. You need to paste this text and execute the command in a terminal window.

	![](./images/pots/mq-cp4i/lab2/image15.png)
	
1. Return to the terminal window or open a new window. Paste the contents into the window and hit enter. 

	![](./images/pots/mq-cp4i/lab2/image16.png)
	
1. Return to the Platform Navigator and close the registration request pop-up window. Notice that the status has now changed to *Processed* and the hyperlink changed from Approve to Reprocess.

	![](./images/pots/mq-cp4i/lab2/image17.png)
	
1. This only needs to be done once. When the request has been approved and the secret created in the *mq* namespace, all other queue managers will be deployed without this error. 
	
	Click the *IBM Cloud Pak for Integration* hamburger menu on the left side bar then select **Integration Home** to return to the *Platform Navigator*. 
	
	![](./images/pots/mq-cp4i/lab2/image18a.png) 

<a name="mqconsole"></a>	
### Start MQ Console

1. In the navigator under *Runtimes* and your queue manager shows a *Ready* status, you can now click the hyperlink to open the MQ console for your queue manager. The runtime instance shows *mq00-quickstart-cp4i*, but that is not your queue manager name. You named it **mq00qs**.

	![](./images/pots/mq-cp4i/lab2/image18b.png)
	
	You receive a potential security risk warning. Click *Advanced* then *Accept the risk* as you did before.

	![](./images/pots/mq-cp4i/lab2/image107.png)

The MQ Console looks nothing like MQ Explorer. It doesn't even look like earlier versions of MQ Console. Feel free to poke around (or click around) and explore the various tiles and side bar menus. When you are ready, continue to the next step - creating a queue.

1. 	Click the *Create a queue* tile. 

	![](./images/pots/mq-cp4i/lab2/image108.png) 

1. A number of tiles are displayed for the different queue types. Behind each tile are the properties for that particular queue type. Click the tile for a *Local* queue. 

	![](./images/pots/mq-cp4i/lab2/image20.png) 
	
1. Only the required basic options are displayed with the required field *Queue name* indicated by an asterick. If you need to alter or just want to see all available options, you can click the button under *Show all available options*. For now, just enter the name for the test queue **app1** and click *Create*.

	![](./images/pots/mq-cp4i/lab2/image21.png) 
	
1. An MQ channel needs to be defined for communication into MQ. Click the *Manage* option.

	![](./images/pots/mq-cp4i/lab2/image110.png)
	
1. Select the *Communication* tab, click *App channels*, then click *Create +*.

	![](./images/pots/mq-cp4i/lab2/image109.png)
	
1. Read the definition, then click *Next*.

1. To keep it simple enter the name of your queue manager (mq00qs for the instructor) so that the channel name matches the queue manager name. Click *Create*.

	![](./images/pots/mq-cp4i/lab2/image111.png)
	
1. By default MQ is secure and will block all communication without explicit configuration. We will allow all communication for the newly created channel. Click on *Configuration* in the top right corner:
    
    ![](./images/pots/mq-cp4i/lab2/image112.png)
    
1. Click the *Security* tab, *Channel authentication* section, then click *Create +*.
    
    ![](./images/pots/mq-cp4i/lab2/image113.png)   
    
1. We will create a *channel auth* record that blocks nobody and allows everyone. Select **Block** from the pull down, and click the *Final assigned user ID* tile. 

	![](./images/pots/mq-cp4i/lab2/image114.png)   
	
1. For *Channel name* enter the channel name you just created. Scroll down and type  **nobody** in the *User list* field, select *nobody*, then click the "+" sign to add it.

	![](./images/pots/mq-cp4i/lab2/image115.png)

	Note: You must click "+" after entering the name.  	
	![](./images/pots/mq-cp4i/lab2/image116.png)
	
	You will receive a succes notification.
	
	![](./images/pots/mq-cp4i/lab2/image117.png)
	
## Test MQ

MQ has been deployed within the Cloud Pak for Integration to other containers deployed within the same Cluster. This deployment is NOT accessible externally. Depending on your scenario you can connect ACE / API Connect / Event Streams, etc to MQ using the deployed service. This acts as an entry point into MQ within the Kubernetes Cluster. Assuming you followed the above instructions within the deployment the hostname will be of the form mq00qs-cp4i-ibm-mq. To verify the installation we will use an MQ client sample within the deployment. 

1. Return to the OCP Console. Make sure you are in the MQ namespace, by clicking *Projects*, scrolling to find *cp4i-mq* and clicking its hyperlink. 

	![](./images/pots/mq-cp4i/lab2/image118.png)
	
1. 	Once in the *cp4imq* namespace, click the drop-down for *Workloads* then select *Pods*. You will see the one pod for your MQ instance. You will see that the *Status* is **Running** and there are three containers running in this pod. You will also see a pod for registering your instance with the Operations Dashboard (tracing) that has been completed
 
	![](./images/pots/mq-cp4i/lab2/image119.png)
	
1. Click the hyperlink for the pod. Now you will see quite a bit of details about the pod. Explore the details where you find graphs for memory and cpu usage, filesystem and network. Scroll down and you will find the containers and volumes. From this panel you can drill down into any of these. But for now, you need to run a test. Click the *Terminal* tab which will automatically log you into the Queue Manager container.

	![](./images/pots/mq-cp4i/lab2/image31.png)
	
1. Run the following commands to send a message to the **app1** queue. Don't forget to replace 00 with your student ID.

	```
	export MQSERVER='mq00qs/TCP/mq00-quickstart-cp4i-ibm-mq(1414)' 
	```
	
	The format of this environment variable is :
	*channel name / TCP / host name (port)*
	
	The host name is actually the network service for your queue manager instance.
	
	```
	/opt/mqm/samp/bin/amqsputc app1 mq00qs
	```
	
	This command is putting a message on queue **app1** on queue manager **mq00qs**.
	Type a message such as:'
	
	```
	sending my first test message to qm mq00qs queue app1
	```
	
	Hit enter again to end the program.

	![](./images/pots/mq-cp4i/lab2/image120.png)
	
1. Return to the MQ Console and navigate back to the queue manager view by clicking on *Manage*. 

	![](./images/pots/mq-cp4i/lab2/image121.png)

1. Notice the **app1** queue has a queue depth of 1 with a maximum queue depth of 50,000. Select the *app1* queue. 

	![](./images/pots/mq-cp4i/lab2/image122.png)
	
1. Click the hyperlink for queue **app1**. Here you see the messages on the queue. You will recognize your message under *Application data* along with application name and time stamp.

	![](./images/pots/mq-cp4i/lab2/image123.png)

	
Congratulations! on completing Lab 2.

## Cleanup

1. Return to the Platform Navigator. Under *Runtimes* find your instance, click the elipsis on the right and select **Delete**. This will delete the queue manager pods and all related artifacts. This will help reduce load on the cluster as you continue the rest of the labs. This queue manager will not be needed again.

	![](./images/pots/mq-cp4i/lab2/image35.png)
	
1. You will be required to enter the name of your instance. Then the *Delete* button becomes active. You must now click the *Delete* button.

	![](./images/pots/mq-cp4i/lab2/image124.png)
	
1. Your instance is now gone and you will receive a success pop-up.

	![](./images/pots/mq-cp4i/lab2/image125.png)
   
[Continue to Lab 3](mq_cp4i_pot_lab3.html)

[Return MQ CP4I Menu](mq_cp4i_pot_overview.html)