---
title: Creating an MQ Instance Using the Platform Navigator
toc: false
sidebar: labs_sidebar
folder: pots/mq-cp4i
permalink: /mq_cp4i_pot_lab1.html
summary: Create an MQ Queue Manager from Navigator
applies_to: [administrator,developer]
---

# Lab 1 - Creating an MQ Instance Using the Platform Navigator

Starting with this lab, each attendee will be assigned an ID number (01 - 30) by the instructor if running a PoT. If running this lab independently, use ID number 00. During a PoT the instructor will always be ID 00.

These instructions assume you are using the DTE Virtual Desktop Image (VDI) for the *MQ on CP4I PoT*.

## Getting Started with MQ on Cloud Pak for Integration 

These instructions document how to setup MQ within Cloud Pak for Integration which is accessible from within the OpenShift Cluster. The instructions have been created using a a Red Hat OpenShift environment deployed on bare metal servers on IBM Cloud however the processa should be similar on other environments.

## Deploying IBM MQ for internal consumers

1. You should have already connected to the OpenShift cluster via the URL provided in your student email. Click *Projects* and scroll down to find the *cp4i* project. Click the hyperlink to select the project.

	![](./images/pots/mq-cp4i/lab1/image202.png)	
1. Scroll down again to find *Networking* on the left side bar menu. Click the drop-down for Networking and select *Routes*. Type *integration* in the Filter field. Click the URL hyperlink for the **integration-navigator-pn** route. 

	![](./images/pots/mq-cp4i/lab1/image203a.png)

1. When prompted with the potential security risk, click *Advanced*. 

	![](./images/pots/mq-cp4i/lab1/image204.png)
	
1. Scroll down and click the *Accept the Risk and Continue* button.

	![](./images/pots/mq-cp4i/lab1/image205.png)

1. If prompted again, respond as you did above. Finally you are routed to the *IBM Cloud Pak Admistration Hub*. Click the *Enterprise LDAP* hyperlink. 

	![](./images/pots/mq-cp4i/lab1/image206.png)

1. Enter the user name and password that was provided in your email to log on.

1. The *IBM Automation* page appears. This is also referred to as the *Integration Platform Navigator*. Click the up arrow on right side of page to close the upper half saving screen space.

	![](./images/pots/mq-cp4i/lab1/image207.png)
	
1. Click *Create runtime* to display the various runtimes available in CP4I.

	![](./images/pots/mq-cp4i/lab1/image208.png)
	
1. A number of tiles are displayed. Click the *Messaging* tile, then click *Next*.

	![](./images/pots/mq-cp4i/lab1/image209.png)
	
1. You will use the option called **Quick start** which will deploy an MQ container with 1 cpu, 1 GB of memory, and measured at 0.25 vpc (Virtual Processor Core). Click the *Quick start* tile then click *Next*.

	![](./images/pots/mq-cp4i/lab1/image210.png)
	
1. On the *Create queue manager* configuration page, enter "mq" plus your student number as a prefix to the Name *quickstart-cp4i* and use the drop-down under *Namespace* to select **mq00**. Click the License acceptance button to turn it on.

	![](./images/pots/mq-cp4i/lab1/image211.png)
	
1. Scroll down to *Queue Manager* section. Using the drop-down under *Type of availability* select **SingleInstance**. Under *Type of volume* leave the default **ephemeral**. You have a choice between ephemeral or persistent-claim. Ephemeral means that the queue manager does not maintain state so does not need persistent-storage. 

	![](./images/pots/mq-cp4i/lab1/image104.png)
	
	1. Under *Tracing* click the button to enable tracing and use the drop-down under *Tracing namespace* to select **cp4i-tracing**.

	Do NOT click Create yet. You need to name your queue manager.

	![](./images/pots/mq-cp4i/lab1/image105.png)
	
1.	On the left side bar, click the button under *Advanced settings* to expose more settings. You may need to scroll to find the *Queue Manager* section again. Under *Advanced: Name* field you see the default **QUICKSTART**. Replace this with your queue manager name using your student ID and qs, ie **mq00qs**. 

	Now click *Create*.
	
	![](./images/pots/mq-cp4i/lab1/image212.png)

1. If all entries are valid, you will receive a notification of success in green. Any errors result in a notification in red. Status remains *Pending* while the queue manager is being provisioned. If you click *Pending* you may receive a *Conditions* pop-up. Since you specified *Tracing* in the configuration, you need to register the queue manager to the operations dashboard. Close the pop-up.

	![](./images/pots/mq-cp4i/lab1/image8a.png)

### Register mq queue manager with Operations Dashboard

This only needs to be done once by the first queue manager that is starting. If the *Pending* status changes to *Ready* then registration is complete and you can skip ahead to the *Start MQ Console* section.

[Go to Start MQ Console](#mqconsole)

1.	If the status has changed to *Error* click the *Error* hyperlink. 
	
	![](./images/pots/mq-cp4i/lab1/image9.png)
	
1.	This error occurs because *MQ* has not been registered in the Operations Dashboard in the *tracing* namespace. Close the *Conditions* pop-up then click *IBM Automation* \>  *Administration* \> *Integration runtimes*. 	
	![](./images/pots/mq-cp4i/lab1/image213.png)
	
	If you receive the security warning, click *Advanced* and then *Accept the Risk and continue*.
	
1.	Click the *cp4i-tracing* hyperlink to open the Operations Dashboard. 
	
	![](./images/pots/mq-cp4i/lab1/image214.png)
	
1.	On the *What's new* window, click the "Don't show again" checkbox then close the window. 
	
	![](./images/pots/mq-cp4i/lab1/image215.png)
	
1.	Once the *Operations Dashboard* is open, click the drop-down for *Manage* on the left sidebar. Then click *Registration requests*.

	![](./images/pots/mq-cp4i/lab1/image216.png)
	
1. The request is displayed with a pod name. This may or may not be a pod for your queue manager (notice the pod name prefix) depending on whether your queue manager was the first to be deployed. The status is new and needs to approved. Click the *Approve* hyperlink. 

	![](./images/pots/mq-cp4i/lab1/image217.png)
	
1. Click the copy button within the pop-up. This text is a command to create the missing secret for the tracing namespace. You need to paste this text and execute the command in a terminal window.

	![](./images/pots/mq-cp4i/lab1/image218.png)
	
1. Return to the terminal window or open a new window. Paste the contents into the window and hit enter. 

	![](./images/pots/mq-cp4i/lab1/image219.png)
	
1. Return to the Platform Navigator and close the registration request pop-up window. Notice that the status has now changed to *Processed* and the hyperlink changed from Approve to Reprocess.

	![](./images/pots/mq-cp4i/lab1/image220.png)
	
1. This only needs to be done once. When the request has been approved and the secret created in the *mq00* namespace, all other queue managers will be deployed without this error. 
	
	Click the *IBM Cloud Pak for Integration* hamburger menu on the left side bar then select **Integration Home** to return to the *Platform Navigator*. 
	
	![](./images/pots/mq-cp4i/lab1/image221.png) 

1. Your queue manager now appears in the *Messaging* tile. 

	![](./images/pots/mq-cp4i/lab1/image222.png) 

<a name="mqconsole"></a>	
### Start MQ Console

1. In the navigator under *Integration Runtimes* and your queue manager shows a *Ready* status, you can now click the hyperlink to open the MQ console for your queue manager. The runtime instance shows *mq00-quickstart-cp4i*, but that is not your queue manager name. You named it **mq00qs**.

	![](./images/pots/mq-cp4i/lab1/image223.png)
	
	If you receive a potential security risk warning. Click *Advanced* then *Accept the risk* as you did before.

	The MQ Console looks nothing like MQ Explorer. It doesn't even look like earlier versions of MQ Console. Feel free to poke around (or click around) and explore the various tiles and side bar menus. When you are ready, continue to the next step - creating a queue.

1. 	Click the *Create a queue* tile. 

	![](./images/pots/mq-cp4i/lab1/image224.png) 

1. A number of tiles are displayed for the different queue types. Behind each tile are the properties for that particular queue type. Click the tile for a *Local* queue. 

	![](./images/pots/mq-cp4i/lab1/image225.png) 
	
1. Only the required basic options are displayed with the required field *Queue name*. If you need to alter or just want to see all available options, you can click the *Custom create* tab. For now, just enter the name for the test queue **app1** and click *Create*.

	![](./images/pots/mq-cp4i/lab1/image226.png) 
	
1. An MQ channel needs to be defined for communication into MQ. Click the *Manage* option.

	![](./images/pots/mq-cp4i/lab1/image227.png)
	
1. Select the *Communication* tab, click *App channels*, then click *Create +*.

	![](./images/pots/mq-cp4i/lab1/image228.png)
	
1. Read the definition, then click *Next*.

1. To keep it simple enter the name of your queue manager (mq00qs for the instructor) so that the channel name matches the queue manager name. *Custom create* tab lets you provide detailed properties. Click *Create*.

	![](./images/pots/mq-cp4i/lab1/image229.png)
	
1. You receive a green success message and the channel appears in the list. 

	![](./images/pots/mq-cp4i/lab1/image230.png)	
1. By default MQ is secure and will block all communication without explicit configuration. We will allow all communication for the newly created channel. Click on *View Configuration* in the top right corner:
    
    ![](./images/pots/mq-cp4i/lab1/image231.png)
    
1. Click the *Security* tab, *Channel authentication* section, then click *Create +*.
    
    ![](./images/pots/mq-cp4i/lab1/image232.png)   
    
1. We will create a *channel auth* record that blocks nobody and allows everyone. Select **Block** from the pull down, and click the *Final assigned user ID* tile. 

	![](./images/pots/mq-cp4i/lab1/image233.png)   
	
1. For *Channel name* enter the channel name you just created. Scroll down and type  **nobody** in the *User list* field then click the "+" sign to add it.

	![](./images/pots/mq-cp4i/lab1/image234.png)

1. Click *Create* to add the record.

	![](./images/pots/mq-cp4i/lab1/image235.png)	
	You will receive a green succes notification and the record appears in the list.
	
	![](./images/pots/mq-cp4i/lab1/image236.png)
	
## Test MQ

MQ has been deployed within the Cloud Pak for Integration to other containers deployed within the same Cluster. This deployment is NOT accessible externally. Depending on your scenario you can connect ACE / API Connect / Event Streams, etc to MQ using the deployed service. This acts as an entry point into MQ within the Kubernetes Cluster. Assuming you followed the above instructions within the deployment the hostname will be of the form mq00qs-cp4i-ibm-mq. To verify the installation we will use an MQ client sample within the deployment. 

1. Return to the OCP Console. Make sure you are in the *mqxx* namespace, by clicking *Projects*, typing **mq** in the filter field to find *mqxx* and clicking its hyperlink. 

	![](./images/pots/mq-cp4i/lab1/image237.png)
	
1. 	Once in the *mqxx* namespace, click the drop-down for *Workloads* then select *Pods*. You will see the one pod for your MQ instance. You will see that the *Status* is **Running** and there are three containers running in this pod. You will also see a pod for registering your instance with the Operations Dashboard (tracing) that has been completed.
 
	![](./images/pots/mq-cp4i/lab1/image238.png)
	
1. Click the hyperlink for the pod. Now you will see quite a bit of details about the pod. Explore the details where you find graphs for memory and cpu usage, filesystem and network. Scroll down and you will find the containers and volumes. From this panel you can drill down into any of these. But for now, you need to run a test. Click the *Terminal* tab which will automatically log you into the Queue Manager container.

	![](./images/pots/mq-cp4i/lab1/image239.png)
	
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

	![](./images/pots/mq-cp4i/lab1/image240.png)
	
1. Return to the MQ Console and navigate back to the queue manager view by clicking on *Manage*. 

	![](./images/pots/mq-cp4i/lab1/image241.png)

1. Notice the **app1** queue has a queue depth of 1 with a maximum queue depth of 50,000. Select the *app1* queue. 

	![](./images/pots/mq-cp4i/lab1/image242.png)
	
1. Click the hyperlink for queue **app1**. Here you see the messages on the queue. You will recognize your message under *Application data* along with application name and time stamp.

	![](./images/pots/mq-cp4i/lab1/image243.png)

	
Congratulations! on completing Lab 2.

## Cleanup

1. Return to the Platform Navigator. Under *Runtimes* find your instance, click the elipsis on the right and select **Delete**. This will delete the queue manager pods and all related artifacts. This will help reduce load on the cluster as you continue the rest of the labs. This queue manager will not be needed again.

	![](./images/pots/mq-cp4i/lab1/image244.png)
	
1. You will be required to enter the name of your instance. Then the *Delete* button becomes active. You must now click the *Delete* button.

	![](./images/pots/mq-cp4i/lab1/image245.png)
	
1. Your instance is now gone and you will receive a success pop-up.

	![](./images/pots/mq-cp4i/lab1/image246.png)
   
[Continue to Lab 2](mq_cp4i_pot_lab2.html)

[Return MQ CP4I Menu](mq_cp4i_pot_overview.html)