---
title: Install ICP4I and MQ Operator on OCP
toc: false
sidebar: labs_sidebar
folder: pots/mq-cp4i
permalink: /mq_cp4i_pot_lab1.html
summary: Introduction to Admin Tools
applies_to: [administrator,developer]
---

# Lab 1 - Install ICP4I and MQ Operator on OCP

## Objectives for Lab 1

Instructor led demo to install OpenShift operators on a deployed OpenShift Container Platform cluster.

* Add IBM catalogues to cluster
* Create projects for ICP4I
* Install IBM Common Service Operator
* Install Platform Navigator Operator
* Create ibm-entitlement-key secret in each namespace
* Create Platform Navigator instance
* Create capability and run-time instances

If this lab is part of a PoT, the lab will be instructor-led demonstrating the installation of the required operators for ICP4I. 

If you are running this PoT personally, you will need to complete the environment set-up and this lab successfully as a pre-req to the rest of the labs.

## Explore OpenShift Console 

If you are running this lab as part of a PoT event, the instructor will provide a URL for you to access a Skytap environment. Open a browser and navigate to the URL which will open the Skytap environment. 

If you are running this lab independent of an event, click this link to reserve a demo which will provision an OpenShift Container Platform cluster.  
[Reserve your demo environment](https://bluedemos.com/) 

{% include note.html content="The above link is only a sample and is a placeholder for the actual hyperlink for the demo VM." %}

The Skytap lab environment for the labs is shown below. You will use a RedHat Enterprise Linux (RHEL) VMware image to complete the MQ on CP4I labs. 

1. The control buttons are highlighted for the VM as well as the overall environment. Click the run button as shown below to start the environment. You can pause the VM or the environment at any time. The stop button will shutdown the VM. You should not need to stop the VM while doing this lab. 
    
    ![](./images/pots/mq-cp4i/lab1/image1.png)  

1. Once the VM is powered up and ready, the icon turns green and shows Running. To open a console for the VM, click the monitor icon.

	![](./images/pots/mq-cp4i/lab1/image1a.png)
	
1. A new browser window will open with the Linux desktop. The black tool bar at the top provides a number of functions. You can adjust the screen resolution or use the cut and paste for example. You can hide the toolbar by clicking the up arrow. 

	![](./images/pots/mq-cp4i/lab1/image2.png) 
	
1. Click the desktop then press enter. The user name **ibmuser** appears. Enter **engageibm** as the password and hit *enter* again or click *Unlock*. You are logged in. 

	![](./images/pots/mq-cp4i/lab1/image3.png)

1. The Firefox browser is active. You will see bookmarks for the OpenShift Console and the Cloud Pak Platform Navigator. 

	![](./images/pots/mq-cp4i/lab1/image4.png)
	
1. Since each provisioned cluster has a unique URL, you will need to modify the properties. Right-click *openshift console* then select properties. 

	![](./images/pots/mq-cp4i/lab1/image5.png)
	 
	Enter the URL of your cluster's openshift console in the *Location* field. Then click *Save*.

	![](./images/pots/mq-cp4i/lab1/image6.png)
	
1. Now click the button to make sure it opens the console. You will need to sign-in with your IBM ID to access the cluster. Enter your ID and click *Continue*.

	![](./images/pots/mq-cp4i/lab1/image7.png)
	![](./images/pots/mq-cp4i/lab1/image8.png)

## Install operators

### Create projects

Now you need to create *Projects* (namespaces) for the capabilities and run-times of Cloud Pak for Integration.

1. Click *Home* > *Projects*.

	![](./images/pots/mq-cp4i/lab1/image12.png)
	
1. Click *Create Project*. Enter **cp4i** in the name field then click *Create*.

	![](./images/pots/mq-cp4i/lab1/image14.png)
	
1. The project is created. Click the *Projects* bread crumb to create another project. 

	![](./images/pots/mq-cp4i/lab1/image15.png)
	
1. Click *Create Project* again and enter **cp4i-ace** for the name then click *Create*.

	![](./images/pots/mq-cp4i/lab1/image16.png)
	
1.  Repeat this process to create the following projects:

	* cp4i-apic
	* cp4i-aspera
	* cp4i-mq
	* cp4i-tracing

### Install source catalogs

1. Click *Operators* > *Installed Operators*.

	![](./images/pots/mq-cp4i/lab1/image9.png)

1. Notice there are no operators installed. OpenShift comes with numerous operators from RedHat and many other vendors. You will see these in *OperatorHub*. But first we need to install the IBM Operator catalog where we will find the ICP4I operators. 
	
	Click the *+* sign at the top of the window to open the editor pane.

	![](./images/pots/mq-cp4i/lab1/image10.png)
	
1. Copy and paste the following yaml snippet into the editor pane and click *Create*. This adds the opencloud operators.

	```
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: opencloud-operators
  namespace: openshift-marketplace
spec:
  displayName: IBMCS Operators
  publisher: IBM
  sourceType: grpc
  image: docker.io/ibmcom/ibm-common-service-catalog:latest
updateStrategy:
  registryPoll:
    interval: 45m
    ```
	
	![](./images/pots/mq-cp4i/lab1/image12.png)
	
	Results:
	
	![](./images/pots/mq-cp4i/lab1/image12a.png)	
1. Next add another catalog source that will allow you to load operators for the Cloud Paks. Click the *+* sign again and copy and paste the following YAML snippet, then click *Create*. This addss the IBM operator catalog.


	```
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: ibm-operator-catalog
  namespace: openshift-marketplace
spec:
  displayName: ibm-operator-catalog
  publisher: IBM Content
  sourceType: grpc
  image: docker.io/ibmcom/ibm-operator-catalog
updateStrategy:
  registryPoll: 
    interval: 45m
    ```
	
	![](./images/pots/mq-cp4i/lab1/image13.png)
	
	Results:
	
	![](./images/pots/mq-cp4i/lab1/image13a.png)	

### Install operators from OperatorHub

You will now install the operators for CP4I. You must install them manually in a specific order one operator at a time, in the order listed below.

1. Click *OperatorHub*.

	![](./images/pots/mq-cp4i/lab1/image17.png)
	
	The *OperatorHub* interface is displayed. 
	
	![](./images/pots/mq-cp4i/lab1/image18.png)
		
1. Enter **IBM Cloud Pak for Integration Platform Navigator** in *Filter by keyword...* field. As you type the filter continues to display a reduced set of operators. At any time, you can click the tile for to choose the operator.

	Click the tile for *IBM Cloud Pak for Integration Platform Navigator*. 
	
	![](./images/pots/mq-cp4i/lab1/image19.png) 
	
1. You can read about the operator, then click *Install*.
	
	![](./images/pots/mq-cp4i/lab1/image20.png)

1. On the *Create Operator Subscription* window, leave the defaults and click *Subscribe*.

	![](./images/pots/mq-cp4i/lab1/image21.png)

1. After a few seconds you will see the operators appear on the *Installed Operators* window. Return to *OperatorHub*, search for the next operator, and repeat the process until all are installed.
	
	For each item on the list of operators below:

	* Search for the specified operator on OperatorHub on the OCP UI.
	* Open the desired operator and click Install.
	* Select the scope (all namespaces or a specific namespace) you would like to install the Cloud Pak in. The scope must be the same for every single operator.
	* Click Subscribe.

	It is best to be patient and wait for each operator to receive a *Succeeded* status before going to the next one.
	
	{% include note.html content="IBM Operator for Redis is an exception as its status may alternate between *Installing* and *Succeeded*." %}	
	```
	IBM Automation Foundation Asset Repository
	IBM Cloud Pak for Integration Operations Dashboard
	IBM API Connect
	IBM App Connect
	IBM MQ
	IBM Aspera HSTS
	Red Hat OpenShift Pipelines Operator
	```

1. It will take a few minutes until all of the operators are successful.
	When done, they should look like this:
	
	![](./images/pots/mq-cp4i/lab1/image22.png)
	
	![](./images/pots/mq-cp4i/lab1/image23.png)
	
	![](./images/pots/mq-cp4i/lab1/image24.png)
	
	![](./images/pots/mq-cp4i/lab1/image25.png)

### Create Docker Registry Secret

Before you create any instances of CP4I you must create a secret for your IBM Entitlement key in order to pull the images from the image registry.

1. Double-click the Terminal icon to open a command window. You may also right-click the desktop and select *Open Terminal*.

	![](./images/pots/mq-cp4i/lab1/image29.png)
	
1. In order to enter kubectl commands or oc commands, you must login to the cluster. 

	On the OpenShift Container Platform (OCP) Console, click your *IAM#* id in the top right corner and select *Copy Login Command*.
	
	![](./images/pots/mq-cp4i/lab1/image30.png)

1. Wait for another browser window to open, then click *Display Token*.	
	![](./images/pots/mq-cp4i/lab1/image31.png)
	
1. Copy the value for *Log in with this token*.

	![](./images/pots/mq-cp4i/lab1/image32.png)

1. Paste this value into the terminal window and hit *enter*.

	![](./images/pots/mq-cp4i/lab1/image33.png)
	
	You are now logged into the *default* namespace.
	
1. You can now enter the command to create the secret.

	Enter the following command to create the secret replacing "**your-entitlement-key-goes-here**" with your real entitlement key:
	
	```
	oc create secret docker-registry ibm-entitlement-key --docker-username=cp --docker-password=**your-entitlement-key-goes-here** --docker-server=cp.icr.io --namespace=cp4i
	```
	
	![](./images/pots/mq-cp4i/lab1/image34.png)
	
	{% include note.html content="If you don't know your IBM Entitlement Key, navigate to [IBM Container Library](https://myibm.ibm.com/products-services/containerlibrary) to obtain your Entitlement key " %}
	
1. You can now login to Docker with the following command replacing "**your-entitlement-key-goes-here**" with your entitlement key.

	```
	docker login cp.icr.io --username cp --password **your-entitlement-key-goes-here**
	```
	
	![](./images/pots/mq-cp4i/lab1/image35.png)
	
1. You will need your IBM Entitlement Key secret in each of the namespaces for capabilities and run-times of CP4I. 

	In the terminal window, hit the up key until the "oc create secret" is found.
	
	Change the namespace to **mq**. Then hit *enter*.

	
	![](./images/pots/mq-cp4i/lab1/image36.png)

1. Repeat the above command for each of the following namespaces:

	* cp4i-tracing
	* cp4i-ace
	* cp4i-apic
	* cp4i-aspera
	* cp4i-mq

	![](./images/pots/mq-cp4i/lab1/image37.png)

### 	Create Platform Navigator Instance
		
1. Return the OCP Console and scroll to find the **IBM Cloud Pak for Integration Platform Navigator** in the *Installed Operators* list.

	![](./images/pots/mq-cp4i/lab1/image26.png)
	
1. Click the operator hyperlink. Read the description, then click *Create Instance*.

	![](./images/pots/mq-cp4i/lab1/image27.png)
	
1. You are presented with the YAML configuration for the *Platform Navigator*. You need to make two changes. 

	* Change the *namespace* to **cp4i**.
	
	* Change the license *accept* to **true**.
	
	Click *Create*.
	
	![](./images/pots/mq-cp4i/lab1/image28.png)

   You are notified of success.

1. The deployment may take a couple of minutes. Click the drop-down for *Project* and select **cp4i**.

	![](./images/pots/mq-cp4i/lab1/image38.png)		
1. In the left-hand menu, click Networking > Routes.

	![](./images/pots/mq-cp4i/lab1/image39.png)
	
	Before clicking the route, copy the URL under *Location* and update the FireFox bookmark *Cloud Pak Platform Navigator*.
	
	Click the URL under *Location* to open the Navigator.
	
	![](./images/pots/mq-cp4i/lab1/image40.png)	
1. At the security risk warning panel, click *Advanced*, then click *Accept the Risk and Continue*. 

	![](./images/pots/mq-cp4i/lab1/image41.png)	
1.	Select *Default authentication* for logging in to IBM Cloud Pak Administration Hub. 

	![](./images/pots/mq-cp4i/lab1/image42.png)
	
1. You are presented with a log in screen. Use **admin** for the *Username*. You need the password, so enter this command in your terminal to obtain the password. Copy the returned result and paste into the password field. Then click *Log in*.
	
	```
	oc get secrets -n ibm-common-services platform-auth-idp-credentials -ojsonpath='{.data.admin_password}' | base64 --decode && echo ""
	``` 

	![](./images/pots/mq-cp4i/lab1/image43.png)
	
1. Click *Save* to save the password when prompted by FireFox.

1. The next screen is the CP4I Platform Navigator where you can create CP4I capabilities and run-times. You first want to create a tracing capability so you can track MQ activity in later labs.

	Click *Create capability*.
	
	![](./images/pots/mq-cp4i/lab1/image44.png)
	
1. You are presented with several options. The Operations Dashboard provides the tracing function. Click its tile, then click *Next*.

	![](./images/pots/mq-cp4i/lab1/image45.png)

1. Click the *Development* tile then click *Next*.

	![](./images/pots/mq-cp4i/lab1/image46.png)	
1. On the next panel *Create aan instance of Operations Dashboard*, you will configure the operator. 

	Under *Namespace*, click the drop-down, scroll to the bottom, and select **tracing**.
	
	Click the button under License > Accept to turn it on. 
	
	![](./images/pots/mq-cp4i/lab1/image47.png)
	
	Scroll down, click the drop-down for each storage type and select the appropriate storage class. **ibmc-file-gold-gid** for *Configuration Database storage* and *Shared storage* and **ibmc-block-gold** for *Tracing storage*. Then click *Create*.
	
	![](./images/pots/mq-cp4i/lab1/image48.png)

1. The *Status* will appear *Pending* while storage is being provisioned. Expect 10 minutes to complete.

	![](./images/pots/mq-cp4i/lab1/image49.png)
	
1. Once the status turns to *Ready*, click the hyperlink *cp4i-od-dev* under Name.

	![](./images/pots/mq-cp4i/lab1/image50.png)
	
1. Close the What's New pop-up. Then click *I agree* to accept the license agreement.

	![](./images/pots/mq-cp4i/lab1/image51.png)
	
	The Operations Dashboard then appears. There is nothing to do here yet. You will return to the dashboard once you configure MQ in the next lab.
	
	![](./images/pots/mq-cp4i/lab1/image52.png)
	
Great! You are now ready to start working with MQ in Lab 2. If running a PoT, attendees are now ready to start Lab 2 where they will create queue managers using their assigned IDs.


[Continue to Lab 2](mq_cp4i_pot_lab2.html)

[Return MQ CP4I Menu](mq_cp4i_pot_overview.html)