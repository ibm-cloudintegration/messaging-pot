---
title: Setup Messaging PoT MQ CP4I Lab Environment
toc: true
sidebar: labs_sidebar
folder: pots/mq-cp4i
permalink: /mq_cp4i_pot_envsetup.html
summary: Lab Environment Setup 
applies_to: [developer administrator]
---

# Setup environment for CP4I PoT

## Connect to your Virtual Desktop Image

1. As a CP4I PoT attendee you should have received an email with instructions to connect to a Virtual Desktop Instance which you will use to execute the labs in this PoT. The email provided the details for connecting to you VDI.

	![](./images/pots/mq-cp4i/env-setup/image101.png) 

1. Copy the password by clicking the copy icon. Then click "Open your IBM Cloud Environment". 

	![](./images/pots/mq-cp4i/env-setup/image102.png)
	
1. The VNC window appears. Click connect. 
	
	![](./images/pots/mq-cp4i/overview/image17.png)
	
1. Enter the password included in the details then click *Send Password*.

	![](./images/pots/mq-cp4i/overview/image18.png)
			
1. The Linux desktop appears. You are now logged in as *student*. You'll notice the toolbar on the left. This is used to control the VNC settings. You will use it mostly for copy/paste from outside the VNC desktop. You can hide the toolbar by clicking the arrow on the side of the toolbar.
	
	![](./images/pots/mq-cp4i/overview/image19.png)

## Connect to the Red Hat OpenShift cluster

Your attendee email also included a URL to connect to the Red Hat OpenShift cluster where you will execute the labs. The name of the cluster will vary depending on which one has been reserved. The example below shows the Obi-Wan cluster. 

 You will also receive a userid to use throughout the PoT. Your email will include the name of the cluster and userid. The userids are defined in an LDAP for the cluster. For example if Obi-Wan is the cluster your email will specify Obi-Wan/obiwanxx where xx is your student number. If the cluster is *chopper*, you will receive  **Chopper/chopperxx**. 
 
  The userid will also be the assigned name of a project (namespace) for you to work in. The project will be yours alone throughout the PoT. No one else will have access to your project and you will not have access to any other student's project. 
  
{% include important.html content="Project cp4i-mq was the project name used to document the labs. This is the default namespace for MQ in the COC clusters and was used for documenting since each PoT may be on various clusters. " %} 

1. From your email, copy the URL for the OCP Console.

	![](./images/pots/mq-cp4i/env-setup/image103.png)

1. Paste the URL onto the clipboard of the VDI. Then copy/paste the URL into a browser tab and hit enter. 

	![](./images/pots/mq-cp4i/env-setup/image104.png)	
1. You are taken to the *Overview* page on the OCP console.

	![](./images/pots/mq-cp4i/env-setup/image105.png)

	You will be in your assigned namespace.

	{% include note.html content="IBM Only - If you are running the lab in your own ROKS cluster, you may create a new project (namespace). [Click here to create your namespace](#namespace)" %} 

<a name="download"></a>	
## Download artifacts for MQ on CP4I PoT

You should be logged on your VDI as *student*. 

1. Open a Firefox browser tab and navigate to [Github MQonCP4i](https://github.com/ibm-cloudintegration/mqoncp4i).

	![](./images/pots/mq-cp4i/env-setup/image108.png)
	
1. Click *Code* and select *Download zip*.

	![](./images/pots/mq-cp4i/env-setup/image109.png)

1. Click *Save file* radio button then click *OK*.

	![](./images/pots/mq-cp4i/env-setup/image110.png)
	
1. Open a terminal window by double-clicking the icon on the desktop.

	![](./images/pots/mq-cp4i/env-setup/image111.png)
	
1. Enter the following command to see the zip file you just downloaded.

	```
	cd Downloads
	```
	
1. Enter the following command to unzip the downloaded file:

	```
	unzip mqoncp4i-main.zip
	```
	
	![](./images/pots/mq-cp4i/env-setup/image112a.png)
	
1. Move the unzipped directory to your home directory with the following command:
	
	```
	cd mqoncp4i-main
	```
	
	```
	mv MQonCP4I/ ~/
	```
	
	![](./images/pots/mq-cp4i/env-setup/image113a.png)	
	This will create the directory **/home/student/MQonCP4I**.
	
	![](./images/pots/mq-cp4i/env-setup/image114a.png)
	
Great! You are now ready to start working with MQ in the labs. If running a PoT, attendees are now ready to start Lab 1 where they will create queue managers using their assigned IDs.

[Continue to Lab 1](mq_cp4i_pot_lab1.html)

[Return MQ CP4I Menu](mq_cp4i_pot_overview.html)

<a name="namespace"></a>
## IBM Only - Create Project (namespace) for your lab assets

1. Click *Projects* then click *Create project*.

	![](./images/pots/mq-cp4i/env-setup/image106.png)
	
1. Select a student number between 01 - 99. Append this number to "mq" to create your unique project name within the cluster (**mq00** is reserved for the instructor and has been used to document the lab). Project is the term for namespace in OpenShift. This is used to keep your assets separate from the other students.

	Enter the name of your project **mqxx** in the name field and click *Create*. 
	
	![](./images/pots/mq-cp4i/env-setup/image107.png)
	
### Create entitlement secret for your namespace

1. In OCP console, click the drop-down next to your username and select "Copy Login Command".

	![](./images/pots/mq-cp4i/env-setup/image115.png)

1. A new browser tab opens. Click the *Display Token* hyperlink.

	![](./images/pots/mq-cp4i/env-setup/image116.png)
	
1. Copy the command under "Log in with this token".

	![](./images/pots/mq-cp4i/env-setup/image117.png)
	
1. Open a terminal window and paste the command into the terminal and hit enter which logs you into the cluster. Enter the following command to change to your namespace. Make sure to substitute 00 with your student number.

	```
	oc project mq00
	```

	![](./images/pots/mq-cp4i/env-setup/image118.png)
	
	{% include note.html content="An entitlement key will be provided to you as part of the PoT. If you already have an entitlement key you may use it." %}
		
1. Enter the following command to create the secret replacing "**your-entitlement-key-goes-here**" with your real entitlement key:
	
	```
	oc create secret docker-registry ibm-entitlement-key --docker-username=cp --docker-password=**your-entitlement-key-goes-here** --docker-server=cp.icr.io --namespace=mq00
	```
	
1. You can now login to Docker with the following command replacing "**your-entitlement-key-goes-here**" with your entitlement key.

	```
	docker login cp.icr.io --username cp --password **your-entitlement-key-goes-here**
	```
	
	![](./images/pots/mq-cp4i/env-setup/image119.png)

[Return to lab guide to continue](#download)