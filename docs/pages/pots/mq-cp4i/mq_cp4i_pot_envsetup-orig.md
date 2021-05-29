---
title: Setup Messaging PoT MQ CP4I Lab Environment
toc: false
sidebar: labs_sidebar
folder: pots/mq-cp4i
permalink: /mq_cp4i_pot_envsetup.html
summary: Lab Environment Setup 
applies_to: [developer administrator]
---

# Lab Environment 

The following instructions are for IBMers to provision a RedHat OpenShift Kubernetes Service (ROKS) cluster. The provisioned cluster can then be used for individual practice or for an MQ on CP4I PoT.

## Provision the OpenShift Cluster

1. Open a web browser and navigate to [IBM Demos](https://www.ibm.com/demos).

1. If not signed-in, sign-in with your IBM ID.

1. Scroll down to *Featured Collections* and click *Cloud Pak for Integration*.

	![](./images/pots/mq-cp4i/env-setup/image2.png)

1. Scroll down to *Demo Environments*. Locate *Cloud Pak for Integration 2020.3.1 on ROKS 4.4 Cluster (Custom)*. Click *Reserve Instance*.

	![](./images/pots/mq-cp4i/env-setup/image3.png)

1. Enter a CRM Opportunity number if you have one. 
Click the drop-down for *Purpose* and select the one appropriate. Click the drop-down for *Preferred Geography* and select one near your location. Change the date and time under *Provision until* to provide enough time to complete the labs (maximum is two weeks).  
   Click the checkbox for *I'm not a robot*.
   Then click *Create*.
   
   ![](./images/pots/mq-cp4i/env-setup/image4.png)
   
1. You are notified that your reservation is successful.

	![](./images/pots/mq-cp4i/env-setup/image5.png)
	
1. A tile will appear showing that the cluster is *Provisioning*. The status is grey until it is complete.

	![](./images/pots/mq-cp4i/env-setup/image6.png)
	
1. You also will receive an email that your cluster is being provisioned.

	![](./images/pots/mq-cp4i/env-setup/image7.png)
	
1. It will take 20 - 40 minutes to provision the cluster. This is why the provisioning must be done prior to the start of the PoT. 

	When the provisioning is complete, the button in the tile will turn green with a *Ready* status. At this time you can click the *Open* button to get the Overview of the cluster.
	
	![](./images/pots/mq-cp4i/env-setup/image8.png)
	
1. An *Overview* of the cluster appears showing the status and health of the cluster. Review the buttons on the left to see how the cluster was provisioned. When ready you can click the *OpenShift web console* button to open the console.
	
	![](./images/pots/mq-cp4i/env-setup/image9.png)

You are now ready for Lab 1.
    
[Continue to Lab 1](mq_cp4i_pot_lab1.html)

[Return MQ CP4I Menu](mq_cp4i_pot_overview.html)
