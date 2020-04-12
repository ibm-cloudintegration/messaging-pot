---
title: Event Streams Geo-Replication
toc: false
sidebar: labs_sidebar
folder: pots/msghub
permalink: /msghub_pot_lab8.html
summary: Event Streams Geo-Replication
applies_to: [developer,administrator]
---

# Event Streams Geo-Replication
In this lab exercise you will learn about the geo-replication feature of Event Streams.

## Introduction 

You can deploy multiple instances of IBM Event Streams and use the included geo-replication feature to synchronize data between your clusters that are typically located in different geographical locations. The geo-replication feature creates copies of your selected topics to help with disaster recovery.

Geo-replication can help with various service availability scenarios, for example:

* Supporting your high availability and disaster recovery plans: you can set up geo-replication to support your high availability and disaster recovery architecture, enabling the switching to other clusters if your primary ones experience a problem.

* Making mission-critical data safe: you might have mission-critical data that your applications depend on to provide services. Using the geo-replication feature, you can back up your topics to several destinations to ensure their safety and availability.

* Migrating data: you can ensure your topic data can be moved to another deployment, for example, when switching from a test to a production environment.

	{% include note.html content="Geo-replication is only available in the paid-for version of IBM Event Streams (not available in the Community Edition)." %}

### How it works

The Kafka cluster where you have the topics that you want to make copies of is called the “origin cluster”.

The Kafka cluster where you want to copy the selected topics to is called the “destination cluster”.

So, one cluster is the origin where you want to copy the data from, while the other cluster is the destination where you want to copy the data to.

{% include important.html content="If you are using geo-replication for purposes of availability in the event of a data center outage or disaster, you must ensure that the origin cluster and destination cluster are installed on different systems that are isolated from each other. This ensures that any issues with the origin cluster do not affect the destination cluster." %}

Any of your IBM Event Streams clusters can become destination for geo-replication. At the same time, the origin cluster can also be a destination for topics from other sources.

Geo-replication not only copies the messages of a topic, but also copies the topic configuration, the topic’s metadata, its partitions, and even preserves the timestamps from the origin topic.

After geo-replication starts, the topics are kept in sync. If you add a new partition to the origin topic, the geo-replicator adds a partition to the copy of the topic on the destination cluster to maintain the correct message order on the geo-replicated topic. This behavior continues even when geo-replication is paused.

You can set up geo-replication by using the IBM Event Streams UI or CLI. When replication is set up and working, you can switch to another cluster when needed.

### What to replicate

What topics you choose to replicate and how depend on the topic data, whether it is critical to your operations, and how you want to use it.

For example, you might have transaction data for your customers in topics. Such information is critical to your operations to run reliably, so you want to ensure they have back-up copies to switch to when needed. For such critical data, you might consider setting up several copies to ensure high availability. One way to do this is to set up geo-replication of 5 topics to one destination cluster, and the next 5 to another destination cluster, assuming you have 10 topics to replicate. Alternatively, you can replicate the same topics to two different destination clusters.

Another example would be storing of website analytics information, such as where users clicked and how many times they did so. Such information is likely to be less important than maintaining high availability for your operations, and you might choose not to replicate such topics, or only replicate them to one destination cluster.


### Planning for geo-replication

Consider the following when planning for geo-replication:


* If you want to use the CLI to set up geo-replication, ensure you have the IBM Event Streams CLI installed.

* Prepare your destination cluster by setting the number of geo-replication workers.
    
* Identify the topics you want to create copies of. This depends on the data stored in the topics, its use, and how critical it is to your operations.

* Decide whether you want to include message history in the geo-replication, or only copy messages from the time of setting up geo-replication. By default, the message history is included in geo-replication. The amount of history is determined by the message retention option set when the topics were created on the origin cluster.
    
* Decide whether the replicated topics on the destination cluster should have the same name as their corresponding topics on the origin cluster, or if a prefix should be added to the topic name. The prefix is the release name of the origin cluster. By default, the replicated topics on the destination cluster have the same name.


![](./images/pots/msghub/lab8/image1.png)

## Step 1: Log in to IBM Cloud Private
Since IBM Event Streams (IES) runs on IBM Cloud Private (ICP) you will need to log into ICP to start the IES admin web UI. To demo IES geo-replication, you will need to log into two different ICP clusters. Both ICP clusters are running on Softlayer. In order to keep them separate, we suggest you use Google Chrome browser for one cluster and Firefox browser for the other. Here's a suggestion and a little table in case you forget which is which later. 

{% include note.html content="In order to keep the ICP clusters separate, we suggest you use Google Chrome browser for one cluster and Firefox browser for the other. Here's a suggestion and a little table in case you forget which is which later. " %}

| ICP Master Node | Browser |  IES Cluster        | Helm Release |
|:---------------:|:-------:|:-------------------:|:------------:|
| 169.62.153.77   | Firefox | Source Cluster      | esgeorep     |
| 169.62.251.198  | Chrome  | Destination Cluster | georep       |

### Use Firefox browser for the first ICP / Event Streams cluster	
Use Firefox for the first IES cluster which will be the source cluster where you will create a topic to be replicated.

1. Open a Firefox browser from the icon on the sidebar.

	![](./images/pots/msghub/lab8/image1a.png)
	
1. Navigate to https://169.62.153.77:8443 to open the ICP user interface (UI). 

	![](./images/pots/msghub/lab8/image3a.png)
	
1. Enter **admin / Trb4Ibm2** for the user and password. Click *Log in*.

	![](./images/pots/msghub/lab8/image4.png)
	
### Use Google Chrome browser for the second ICP / Event Streams cluster 

Use Chrome for the second IES cluster which will be the destination cluster where you will replicate topics to.

1. Open a Chrome browser from the icon on the sidebar. 
	
	![](./images/pots/msghub/lab8/image1.png)
	
1. Click **X** to dismiss the warning about certificates.

	![](./images/pots/msghub/lab8/image2.png)	
1. Navigate to https://169.62.251.198:8443 to open the ICP user interface (UI). 

	![](./images/pots/msghub/lab8/image3a.png)

1. Enter **admin / admin** for the user and password. Click *Log in*.

	![](./images/pots/msghub/lab8/image4.png)
	
## Step 2: Configure Geo-replication topic

1. Switch to Firefox (source cluster). 

1. It is a good idea to synchronize the Helm Repositories to get the latest Helm charts before installing the Event Streams Helm chart. Click the hamburger icon to open the menu.

	![](./images/pots/msghub/lab8/image8a.png)

1. Expand *Manage* and select **Helm Repositories**.

	![](./images/pots/msghub/lab8/image8b.png)
 
1. Click *Sync repositories*.

	![](./images/pots/msghub/lab8/image8c.png)

1. Click *OK* to confirm.	

	![](./images/pots/msghub/lab8/image8d.png)
	
1. You will receive a pop-up indicating the sync is in progress. Click *Catalog* on the toolbar.

	![](./images/pots/msghub/lab8/image8e.png)

1. Type **ibm-event** in the search field, then click *ibm-eventstreams-prod* Helm chart.

	![](./images/pots/msghub/lab8/image8.png)

1. Click *Configuration* or the *Configure* button.

	![](./images/pots/msghub/lab8/image9.png)
	
1. Enter **esgeorep** for the *Helm release name*, **es** in the *Target namespace* field, and **regcred** in the *Image pull secret*. Make sure to check the *License* checkbox.

	![](./images/pots/msghub/lab8/image10.png)
	
1. Scroll down, and delete the trailing "/" mark of the URL in the *Docker image registry*.

	![](./images/pots/msghub/lab8/image11.png)
	
	![](./images/pots/msghub/lab8/image12.png)
	
1. Scroll down to the bottom. Make sure that *Enable message indexing* is checked. Enter **2** in the *Geo-replicator workers*. This indicates the number of worker nodes that will perform the replication of messages to a destination node. 
	
	Click *Install*.

	![](./images/pots/msghub/lab8/image13.png)
	
1. Scroll to the top of the window to see the message stating that the *Chart deployment is in progress...*. Click the "hamburger" icon to open ICP menu. 

	![](./images/pots/msghub/lab8/image14.png)
	
1. Expand *Workloads* and select *Helm Releases*. 

	![](./images/pots/msghub/lab8/image16.png)
	
1. You may need to refresh the screen once or twice until you see the *esgeorep* Helm Release.

	![](./images/pots/msghub/lab8/image17.png)
	
1. Click the menu icon again, expand *Workloads*. This time select *Deployments*. The new deployments for georeplab should be at the top. Notice the deployment for replication. Click the *esgeorep-ibm-es-ui-deploy* to open the deployment. 

	![](./images/pots/msghub/lab8/image18.png)
	
1. You will see the NodePort under *Expose details - Ports*. Yours may be different than the screenshot. This is the port for the admin webpage. You may want to record this as you may need it later. Click *Launch* to open the UI. 

	![](./images/pots/msghub/lab8/image19.png)
	
1. A new browser tab opens. If you receive the "Your connection is not secure" message, click *Advanced*.

	![](./images/pots/msghub/lab8/image19a.png)
	
	Click *Add Exception* on the "invalid security certificate" pop-up.
	
	![](./images/pots/msghub/lab8/image19b.png)
	
	Click *Confirm Security Exception*.
	
	![](./images/pots/msghub/lab8/image19c.png)
	
1. A new tab is opened in your browser with the ICP master node URL + the NodePort. If the page remains in a *Loading...* state, click the **X** to close the page and enter the address again. It should then load immediately.

	![](./images/pots/msghub/lab8/image20.png)
	
1. You are presented with the ICP login screen again. Enter **admin / Trb4Ibm2** for the user and password. Click *Log in*.

1. You are taken to the admin UI for your Event Streams Helm Release - **esgeorep**. Click *Generate a starter application*. 
	
	![](./images/pots/msghub/lab8/image21.png)

## Step 3: Generate starter application and topic for replication

1. The *Event Streams Toolbox* opens. Click *Generate application*. 

	![](./images/pots/msghub/lab8/image22.png)

1. On the *Starter application* enter **esgeoreptester** for the Application name. Select *Create topic* (default). Enter a topic name, ie. **esgeorep**. Click *Generate*. 
	
	![](./images/pots/msghub/lab8/image23.png)

1. It will take a moment for the Starter application to be generated.  Once you see the status message *The Starter application has been generated* you may complete the steps that are listed on the page.	
	The first step is to download the Starter application that was generated.  Click the *Download* button and save the archive file to the Downloads directory. 
	
	![](./images/pots/msghub/lab8/image24.png)

2. Next you will need to locate and unpack the file that was downloaded.  Open a command prompt window. Create a new subdirectory in the Downloads directory to contain and build the app. Enter the following commands to extract the starter application. If you are asked if you want to replace any files answer **A** for all. 

	```
	cd Downloads
	mkdir esgeorep
	cd esgeorep
	unzip ~/IBMEventStreams_esgeoreptester.zip
	```
	
	![](./images/pots/msghub/lab8/image25.png)
		
1. You will now build the application as you did in Lab 4, with Maven. Maven is a build utility that is used to create Java applications.  It has been installed, along with Java, in the virtual machine for this lab exercise.  The archive that was extracted contains a **pom.xml** file that provides the directions for Maven to generate the Starter application. Use the following command to build and launch the Starter application.

		```
		mvn install liberty:run-server
		```
	
	![](./images/pots/msghub/lab8/image26.png)
	
1. Now that you have built the application, you can launch it by clicking the hyperlink - http://localhost:9080/georeptester.

	![](./images/pots/msghub/lab8/image27.png)

1. Another browser tab opens with a name *Starter App::IBMEventStreams_esgeoreptester*. Click *Launch Consumer*.

	![](./images/pots/msghub/lab8/image28.png)

1. Another browser tab opens with the name *Consumer(0)::IBMEventStreams_esergeoreptester* for the starter application consumer. Click *Start listening for messages*. 

	![](./images/pots/msghub/lab8/image29.png)
	
1. You can see the activity in the command console. The consumer status changes to *Stop listening for messages*. Do not click this. Leave the consumer listening for messages. 

	![](./images/pots/msghub/lab8/image30.png)
	
1. Return to the *Start App...* tab and click *Launch Producer*. 

	![](./images/pots/msghub/lab8/image31.png)
	
1. Yet another browser tab opens with the name *Producer::IBMEventStreams_esgeoreptester*. Notice the payload which the producer will send in the messages. Click *Start producing messages*. 

	![](./images/pots/msghub/lab8/image32.png)
	
1. You can see the partition where the messages are being delivered and the offset of the messages. Leave the producer running so you will have messages to replicate.

	![](./images/pots/msghub/lab8/image33.png)
	
1. Switch to the consumer tab and observe that messages are in fact being delivered and the consumer is reading them. You can see the message offset and the payload. Click on a message to see the details for that particular message.

	![](./images/pots/msghub/lab8/image34.png)
	
## Replicate messages to remote cluster

1. Return to the IBM Event Streams browser tab. Click the Toolbox back link. 

	![](./images/pots/msghub/lab8/image35.png)
	
1. Click *Topics*.

	![](./images/pots/msghub/lab8/image36.png)
	
1. You see the **esgeorep** topic that you created with the starter application. Click the topic to open the message detail.

	![](./images/pots/msghub/lab8/image37.png)
	
1. You are presented with a display of messages that have been produced, a graphical timeline display, and metadata about the messages.

	![](./images/pots/msghub/lab8/image38.png)
	
1. Click the *Topics* back button to return.

1. Click *Geo-replication*.

	![](./images/pots/msghub/lab8/image39.png)
	
1. The Geo-replication panel appears. It is split in two, with the local topics display on the left and the geo-replication information on the right. Once you define a remote cluster, its topics will display on the right.

	![](./images/pots/msghub/lab8/image40.png)
	
### Acquire destination information

In order to replicate messages to a remote cluster, you must acquire the connection information for that cluster.

1. Swith to the Chrome browser (destination cluster). 
	
1. You should have a browser tab already open on the destination ICP cluster at: https://169.62.251.198:8443.

	1. It is a good idea to synchronize the Helm Repositories to get the latest Helm charts before installing the Event Streams Helm chart. Click the hamburger icon to open the menu.

	![](./images/pots/msghub/lab8/image8f.png)

1. Expand *Manage* and select **Helm Repositories**.

	![](./images/pots/msghub/lab8/image8g.png)
 
1. Click *Sync repositories*.

	![](./images/pots/msghub/lab8/image8h.png)

1. Click *OK* to confirm.	

	![](./images/pots/msghub/lab8/image8d.png)
	
1. You will receive a pop-up indicating the sync is in progress. Click *Catalog* on the toolbar.

	![](./images/pots/msghub/lab8/image8i.png)

1. Click *ibm-eventstreams-prod* helm chart.

	![](./images/pots/msghub/lab8/image8j.png)

1. Click *Configuration* or the *Configure* button.

	![](./images/pots/msghub/lab8/image9a.png)
	
1. Enter **georep** for the *Helm release name*, **es** in the *Target namespace* field, and **regcred** in the *Image pull secret*. Make sure to check the *License* checkbox.

	![](./images/pots/msghub/lab8/image9b.png)
	
1. Scroll down, and delete the trailing "/" mark of the URL in the *Docker image registry*.

	![](./images/pots/msghub/lab8/image11.png)
	
	![](./images/pots/msghub/lab8/image12.png)
	
1. Scroll down to the bottom. Make sure that *Enable message indexing* is checked. Enter **2** in the *Geo-replicator workers*. This indicates the number of worker nodes that will perform the replication of messages to a destination node. 
	
	Click *Install*.

	![](./images/pots/msghub/lab8/image13.png)
	
1. Scroll to the top of the window to see the message stating that the *Chart deployment is in progress...*. Click the "hamburger" icon to open ICP menu. 

	![](./images/pots/msghub/lab8/image14a.png)
	
1. Expand *Workloads* and select *Helm Releases*. 

	![](./images/pots/msghub/lab8/image16.png)
	
1. You may need to refresh the screen once or twice until you see the *georep* Helm Release.

	![](./images/pots/msghub/lab8/image17.png)
	
1. Click the menu icon again, expand *Workloads*. This time select *Deployments*. The new deployments for georeplab should be at the top. Notice the deployment for replication. Click the *georep-ibm-es-ui-deploy* to open the deployment. 

	![](./images/pots/msghub/lab8/image18a.png)
	
1. You will see the NodePort under *Expose details - Ports*. Yours may be different than the screenshot. This is the port for the admin webpage. You may want to record this as you may need it later. Click *Launch* to open the UI. 

	![](./images/pots/msghub/lab8/image18b.png)
	
1. A new browser tab opens. If you receive the "Your connection is not secure" message, click *Advanced*. Confirm anything the security exceptions. The screens differ slightly from Firefox, but the process is pretty much the same.
		
1. A new tab is opened in your browser with the ICP master node URL + the NodePort. If the page remains in a *Loading...* state, click the **X** to close the page and enter the address again. It should then load immediately.

	![](./images/pots/msghub/lab8/image20.png)
	
1. You are presented with the ICP login screen again. Enter **admin / admin** for the user and password. Click *Log in*.

1. Click *Topics*.

	![](./images/pots/msghub/lab8/image47a.png)
	
1. There may or may not be topics in the topic list. You can ignore any that are there. Click *Geo-replication*./ 

	![](./images/pots/msghub/lab8/image48.png)
	
1. You will see the split screen geo-replication menu with the local topics on the right and any remote topics on the right under *Geo-replication*. Again, ignore any references to other clusters or topics as this is the common replication site for the Event Streams PoT. 

	Read the notes on the page. You want to replicate messages to this cluster, so you need the connection information to route to this cluster. Click the *Generate connection information for this cluster* button. 
	
	![](./images/pots/msghub/lab8/image49.png)
	
1. Don't blink because you may miss a message that the connection information is being generated. Once generated, you need to copy the information. Click *Copy connection information* button.

	![](./images/pots/msghub/lab8/image50.png)
	
	You have now copied the connection information to the clipboard and you will receive a green checkmark saying that the info is copied.
	
1. Return to the other Firefox browser, where you have the soure Event Streams cluster. Click *Add destination cluster*. 

	![](./images/pots/msghub/lab8/image51.png)
	
1. Paste the copied connection information in the box provided.

	![](./images/pots/msghub/lab8/image52.png)
	
1. Event Streams immediately tries to validate the information. If correct, you will receive green checkmarks. Click *Add destination folder*. 

	![](./images/pots/msghub/lab8/image53.png)
	
1. Remember, it was stated previously that the remote cluster is shown on the right. So you see the Helm Release **georep** on the remote cluster is ready for geo-replication with two workers assigned, but no topics yet. 

	On the left, you see your local Helm Release **esgeorep** with available topics for replicattion. **esgeorep** is the only one available since it is the only topic you created.
	
	Click the check box for **esgeorep** and click the *Geo-replicate to destination* button. 
	
	![](./images/pots/msghub/lab8/image54.png)
	
1. On the right side of the screen, you see that **georep** has received the request and wants you to confirm that it is OK to create topic **esgeorep** on the remote cluster. You also have buttons to add a prefix the destination topic and to include message history.

	![](./images/pots/msghub/lab8/image56a.png)
	
1. Your starter app has been producing messages, so leave *Include message history* checked. Since source cluster may be replicating topics to this cluster, you should add a prefix. Event Streams adds the source cluster name as the prefix. In this case, the cluster name and topic are the same so it produces **esgeorep_esgeorep**.

	Click *Create*. 
	
	![](./images/pots/msghub/lab8/image56a.png)
	
1. The status changes to *Creating* on the remote cluster (**georep**). Notice on the topic name now has the prefix of **esgeorep_esgeorep**. The topic on the local cluster retains the original topic nanme **esgeorep** and is marked with a replication icon on the local cluster. 

	![](./images/pots/msghub/lab8/image58a.png)
	
1. On **georep** the status eventually changes to *Assigning*, then *Running*.

	![](./images/pots/msghub/lab8/image66.png)
	
1. Switch to the Chrome browser (destination cluster). You will see a new topic on the local cluster (**georep**) named **esgerep_esgeorep**. It has a right facing arrow indicating that it is being replicated from a remote cluster. As on the original, the topic has two replicas.

	![](./images/pots/msghub/lab8/image60a.png)
	
1. On the right side under Geo-replication, click the down arrow and you will see the original topic **esgeorep**. Clicking the elipsis, you have the option to pause replicators, stop replication, restart replicators, etc. 

	![](./images/pots/msghub/lab8/image63.png)

1. Close the Geo-replication pane by clicking the "X" in ther top right corner. You return to the full screen Topics display. You now see which cluster the topic **esgeorep** is coming from highlighted in green.

	![](./images/pots/msghub/lab8/image64.png)
	
1. Click the topic name **esgeorep_esgeorep** and the messages will be displayed just like they were on the original cluster.

	![](./images/pots/msghub/lab8/image62.png)

1. Switch back to the local cluster on the Firefox browser. In the right pane (remote cluster), click the "Back to geo-replication" button.

	![](./images/pots/msghub/lab8/image65a.png)

1. 	The display shows the topic in the right pane. In the left pane you see the name of the topic with the replication icon and the IP address of the remote cluster where the topic is being replicated.

	![](./images/pots/msghub/lab8/image65.png)

## Clean up the environment

1. On the Firefox browser, click the Producer tab and click the "Stop producing messages" button.

	![](./images/pots/msghub/lab8/image67.png)
	
1. Click the Consumer tab and click the "Stop listening for messages" button.

	![](./images/pots/msghub/lab8/image68.png)
	
1. You should also close the Starter app... tab and close the terminal where the app was running.

	![](./images/pots/msghub/lab8/image69.png)

## Congratulations
You have successfully configured geo-replication and replicated Event Streams messages. 

You have completed Lab 8 of the Event Streams PoT.

[Continue to Lab 9 - Kafka Sink Connector for MQ](msghub_pot_lab9.html)