---
title: Appendix A - Create Event Streams Sample Applications
toc: false
sidebar: labs_sidebar
folder: pots/msghub
permalink: /msghub_pot_appendix_a.html
summary: Learn how to create the Event Streams sample applications to test an Event Streams configuration.
applies_to: [developer,administrator]
---

## Overview

This appendix describes the steps required to create the sample applications that may be used to demonstrate Event Streams operations.  These steps have already been completed in the the lab image, and are provided here for reference only.

## Create the Sample Applications

The IBM Event Streams service provides several test applications that may be used to demonstrate EWvent Streams functions.  The steps in this appendix assume that you are using an **Xubuntu 64-bit 14.04** operating system with a graphical user interface enabled.  

Complete the following steps to install the test applications.  

### Step 1: Install Java

1. Open a terminal window from within the **Xubuntu 64-bit 14.04** image.

	![](./images/pots/msghub/msghub-ibmcloud-testing-1.png)
	
2. The test applications are Java based. To successfully build and execute the applications you need to install a Java development toolkit (JDK) and the **gradle** build utility.  Complete the following steps if Java has not been installed in your Ubuntu image:

	1. Change to your home directory.
	
		```
		cd ~
		```
	
		![](./images/pots/msghub/msghub-ibmcloud-jdk-install-1.png)
		
	2. Enter the following command to add an apt repository that includes the JDK.

		{% include note.html content="You may be prompted to use a different PPA archive.  Press the **ENTER** key to continue with the current archive." %}

		```
		sudo apt-add-repository ppa:webupd8team/java
		```
	
		![](./images/pots/msghub/msghub-ibmcloud-jdk-install-2.png)
		
	3. Enter the following commands to update the **gpg** protection key for Mongo (necessary to avoid an error when running the update command) as well as update the local **apt** repository.

		```
		sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
		sudo apt-get update
		```
		
		![](./images/pots/msghub/msghub-ibmcloud-jdk-install-3.png)
		
	4. Enter the following command to install the JDK.

		```
		sudo apt-get install oracle-java8-installer
		```
		
		![](./images/pots/msghub/msghub-ibmcloud-jdk-install-4.png)
		
	5. You may see several prompts to accept license agreements for Java.  Accept these license agreements to continue.
		
		![](./images/pots/msghub/msghub-ibmcloud-jdk-install-5.png)
		
		![](./images/pots/msghub/msghub-ibmcloud-jdk-install-6.png)
		
	6. Enter the following command once the installation has completed in order to verify that the JDK was successfully installed.

		```
		java -version
		```

		![](./images/pots/msghub/msghub-ibmcloud-jdk-install-7.png)
		
	7. You must now ensure that the default Java installation is set to the downloaded version.  Use the following commands to set the downloaded Java version as the default:

		```
		sudo unlink /usr/lib/jvm/default-java
		sudo ln -s /usr/lib/jvm/java-8-oracle/ /usr/lib/jvm/default-java
		```
	
		![](./images/pots/msghub/msghub-ibmcloud-jdk-install-8.png)

### Step 2 Install Gradle

1. Next you will need to install the **gradle** build utility.  Use the following command to install **gradle**.

	```
	sudo apt-get install gradle
	```

	![](./images/pots/msghub/msghub-ibmcloud-gradle-install-1.png)
	
### Step 3 Create the Sample Applications

1. IBM Cloud provides a github project that contains the Event Streams samples.  Use the following command to download this project:

	```
	git clone https://github.com/ibm-messaging/event-streams-samples.git
	```	

	![](./images/pots/msghub/ES-appendixA-clone.png)

2. Next you will need to use the **gradle** command to build the sample project.  Change to the directory that contains the downloaded project and then build the sample project using the following commands:

	```
	cd event-streams-samples/kafka-java-console-sample/
	gradle clean && gradle build
	```

	![](./images/pots/msghub/ES-appendixA-build.png)
	
3. The Event Streams sample applications are now ready for testing.  Refer to **Lab 1** of the Event Streams Proof of Technology or the Event Streams documentation on IBM Cloud for information on how to use the sample applications.   


