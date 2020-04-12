---
title: Lab Guide
toc: false
sidebar: labs_sidebar
folder: pots/mq-sfdc
permalink: /mq_sfdc_pot_lab_guide.html
summary: MQ Salesforce Bridge Lab Guide
---

# Overview

This Proof of Technology (PoT) provides a hands-on experience for those needing to understand how the IBM MQ Salesforce Bridge may be used to support bidirectional integration of IBM MQ and Salesforce.  Integration is enabled via the Publish and Subscribe capabilities of IBM MQ and the **Platform Event** and **Push Topic** features of Salesforce.  

For this lab you will examine:

* How to configure your IBM MQ and Salesforce environment to enable IBM MQ's pub/sub capabilities to publish events related to create, update and delete operations that have occurred against a Salesforce **Account** object.
* How to use messages published to an IBM MQ Pub/Sub Topic to modify a Salesforce **Account** object.  

Note that the IBM MQ Salesforce Bridge is supported on Linux platforms only.  

## Step 1 Complete All Prerequisites

You must complete the prerequisite steps for this PoT before starting this lab.  The prerequisite steps may be found at the following URLs:

* [MQ Salesforce Bridge - Prerequisites](https://pages.github.ibm.com/cloudintegration/PoT-messaging/mq_sfdc_pot_prereqs.html)
* [MQ Salesforce Bridge - Create Salesforce Developer Account](https://pages.github.ibm.com/cloudintegration/PoT-messaging/mq_sfdc_pot_setup_salesforce.html)

## Step 2 Prepare the Salesforce Environment

In order to demonstrate how integration between IBM MQ and Salesforce is enabled it will be necessary to configure the following for your Salesforce account:

* An X.509 self-signed certificate and keystore will need to be created for TLS protection of messages sent between IBM MQ and Salesforce. This keystore will need to be transferred to the Virtual Machine that is hosting MQ.  Once that has been done the MQ Salesforce Bridge will need to be configured to use this keystore.
* A set of “PushTopics” will need to be defined for the Salesforce account in order to enable Salesforce to send event messages whenever an update to an Account record is made.
* A "Platform Event" message and corresponding "Trigger" will need to be defined for the Salesforce account in order to enable IBM MQ to send Platform Events to Salesforce.  The Salesforce Bridge will subscribe to messages that are published to a topic of **/\<root\>/mqtosfb/event/+** (where **\<root\>** is the root topic configured in the Salesforce Bridge) and create Platform Event messages that may be processed by Salesforce.  

The following steps will guide you through the completion of these tasks.

### Creating an X.509 Self-signed Certificate and a Keystore

1. Access the **Xubuntu 64-bit 14.04** Virtual Machine.  If logon is required use the following Username and Password:
	* 	student
	*  Passw0rd! (Where 0 is numeral zero) 
	
	![](./images/pots/mq-sfdc/salesforce_Ubuntu_Login.png)
	
2. Logon to Salesforce by double-clicking on the **Firefox Web Browser** icon to open a web browser. 

	![](./images/pots/mq-sfdc/salesforce_launch_browser.png)
	
3. Enter the following URL and then click on the **LOGIN** link: [https://www.salesforce.com](https://www.salesforce.com)
	
	![](./images/pots/mq-sfdc/salesforce_salesforce_url.png)

4. Enter your Username and Password and then click on the **Log In** button.

	![](./images/pots/mq-sfdc/salesforce_salesforce_log_in.png)
	
5. You may be required to verify your identity.  Follow the instructions to access the verification code from your email and enter it to proceed.  

	![](./images/pots/mq-sfdc/salesforce_verify_identity.png)
	
6. The **Force.com** Home page should appear.  You will need to scroll the page down until you locate the section titled **Security Controls**.  Expand that section and then click on the **Certificate and Key Management** menu item.

	![](./images/pots/mq-sfdc/salesforce_security_controls.png)
	
7. The **Certificate and Key Management** page will appear.  Click on the **Create Self-Signed certificate** button.

	![](./images/pots/mq-sfdc/salesforce_create_cert.png)
	
8. Enter the values as shown in the following screen shot and then click on the **Save** button.
	
	Field Name             | Value to Enter
	:---------------------:|:--------------:
	Label                  | SQM1 Connection
	Unique Name            | SQM1_Connection
	Type                   | Self-Signed
	Exportable Private Key | *Checked*
	    
	![](./images/pots/mq-sfdc/salesforce_cert_entries.png)
	
	
9. Click on the **Back to List:** link to return to the **Certificate and Key Management page**.

	![](./images/pots/mq-sfdc/salesforce_created_cert.png)

10. Click on the **Export to Keystore** button.

	![](./images/pots/mq-sfdc/salesforce_export_keystore.png)
	
11. Assign a keystore password of **passw0rd** (where **0** is numeral zero) and then click on the **Export** button.

	![](./images/pots/mq-sfdc/salesforce_keystore_password.png)

12. A dialog will appear asking you to confirm saving of the file. The format of the dialog may be different depending on the web browser you are using.  Take the appropriate actions to save the file.

	![](./images/pots/mq-sfdc/salesforce_save_keystore.png) 
	
13. Depending on the web browser you are using you may see either a confirmation pop up dialog or you may see an acknowledgement of the download towards the bottom of the browser window.  Click on the appropriate link to locate the downloaded file.

	![](./images/pots/mq-sfdc/salesforce_save_keystore_2.png)

14. Copy the keystore file to the **/home/student** directory. 

	![](./images/pots/mq-sfdc/salesforce_keystore_home.png)

### Create a Salesforce "Platform Event"

Starting from IBM MQ Version 9.0.4 you can use an IBM MQ application to create JSON formatted messages that are published to a queue manager topic of **/\<root\>/mqtosfb/event/+**.   (For this lab you will use a root of **/sf**.)  The IBM MQ Salesforce Bridge subscribes to the topic, gets the content from the messages, and uses the content to publish Salesforce **Platform Event** messages.  

{% include note.html content="**Platform Events** are the event messages (or notifications) that Salesforce and external apps send and receive to take further action." %}

Complete the following steps in order to configure the Salesforce **Platform Event** objects that will be used in this lab. 

1. Click on the drop down icon next to your Salesforce username and then click on the **Setup** menu item.

	![](./images/pots/mq-sfdc/salesforce_setup_menu.png)
 
2. Enter **Platform Events** in the **Quick Find / Search...** field and then click on the **Platform Events** menu item.

	![](./images/pots/mq-sfdc/salesforce_platform_events_menu.png)
	
3. 	Click on the **New Platform Event** button.

	![](./images/pots/mq-sfdc/salesforce_new_platform_event_btn.png)
	
4. Complete each of the fields as shown.  Note that the **Object Name** field will be automatically filled in when you enter the **Label**.  Ensure that you have selected the **Standard Volume** for the event type and have also selected the **Deployed** status, then click on the **Save** button.

	![](./images/pots/mq-sfdc/salesforce_save_platform_event.png)
	
5. Now you need to add two custom fields to the event.  Scroll down to locate the **Custom Fields & Relationships** section and then click on the **New** button.

	![](./images/pots/mq-sfdc/salesforce_new_field_btn.png)
	
6. Scroll down to locate the **Data Type** section and select the **Text** type.  Click on the **Next** button to continue.

	![](./images/pots/mq-sfdc/salesforce_event_type_text.png)
	
7. Complete each of the fields as shown.  Since this field will be used to match the **Account Name** field in the **Accounts** object it will need to be large enough to hold any account name that is passed in the event.  Click on the **Save** button when finished.  

	![](./images/pots/mq-sfdc/salesforce_save_event_field.png)
	
8. Repeat the previous step to create a new field named **Rating**.  Set the **Data Type** to **Text**, the **Length** to **40** and select the **Required** checkbox.  
9. Take note of the **API Name** values that are assigned to the platform event and to each of the custom fields that you have created.  These will be needed later on for testing.

	![](./images/pots/mq-sfdc/salesforce_api_names.png)
	
10. Now you must create a **Trigger** for the **Update\_Account\_Rating** Platform Event.  A trigger is used to take action on platform events that have a **Subscription** associated with them.  Click on the **New** button to create a new Trigger.

	![](./images/pots/mq-sfdc/salesforce_create_trigger.png)
	
11. The following is the Apex code that will be used to process the **Platform Event** that will be sent by the Salesforce Bridge when a message is published to a specific topic on the IBM MQ queue manager where the Bridge has been configured.  Copy and paste this code into the Apex code editor window and then click on the **Save** button.  

	
	{% include note.html content="You will configure the subscription topic string later in this lab." %}
  

	```
	// Trigger to update an Account object with a new Rating value.
	trigger UpdateAccountRatingTrigger on Update_Account_Rating__e (after insert) {
	    
	    try{
	        // Iterate across all events.
	        for (Update_Account_Rating__e event : Trigger.New) {
	            Account acct = [SELECT Rating FROM Account WHERE Name = :event.Account_Name__c];
	            // Update existing records
	            if (acct != null) {
	                acct.Rating = event.Rating__c;
	                update acct;
	            }
	        }
	    } catch(DMLException e) {
	        System.debug('Error when processing UpdateAccountRatingTrigger');
	    }
	}	
	```

	![](./images/pots/mq-sfdc/salesforce_edit_apex_code.png)
	
12. Click on the **Back to List: Custom Object Definitions** link.

	![](./images/pots/mq-sfdc/salesforce_back_link.png)
	
13. Note that your trigger as well as an associated **Subscription** have been created.

	![](./images/pots/mq-sfdc/salesforce_subscription.png)	

<a name="pushtopics"></a>  
### Create Salesforce "Push Topics"

You are now ready to create a set of Salesforce **"Push Topics"** that will generate events whenever there are changes to **Account** objects in your Salesforce instance.  

1. Click on the drop down icon next to your Salesforce username and then click on the **Developer Console** menu item.  

	![](./images/pots/mq-sfdc/salesforce_open_dev_console.png)
	
2. The **Force.com Developer Console** window will appear.  This window enables developers to perform a number of tasks to customize their Salesforce instance.  In our lab we will use the **Execute Anonymous** dialog to enter the necessary commands.   Click on the **Debug** menu item and then click on the **Open Execute Anonymous Window** menu item.

	![](./images/pots/mq-sfdc/salesforce_open_execute_anonymous_window.png)

3. A window titled **Enter Apex Code** will appear.  (Note that this window may contain contents from previous commands that have been entered.)  You will use this window to enter the various commands that will be required to configure the Push Topics.

	![](./images/pots/mq-sfdc/salesforce_enter_apex_window.png)
 

4. Clear the contents of the window, if necessary.  Enter the following **Apex** code to create a new Push Topic that will generate an event for every new Salesforce Account object that is created. Click on the **Execute** button to execute the statements.

	```
	PushTopic pushTopic = new PushTopic();
	
	pushTopic.Name = 'AccountObjectCreates';
	
	pushTopic.Query = 'SELECT Id, AccountNumber, Name, BillingAddress, BillingStreet, BillingCity, BillingState,BillingPostalCode FROM Account';
	
	pushTopic.ApiVersion = 39.0;

	pushTopic.NotifyForOperationCreate = true;
	pushTopic.NotifyForOperationUpdate = false;
	pushTopic.NotifyForOperationDelete = false;
	pushTopic.NotifyForFields = 'All';

	insert pushTopic;
	```
5. Reopen the **Open Execute Anonymous Window**.  Clear all existing statements and enter the following **Apex** code to create a new Push Topic that will generate an event for every existing Salesforce Account object that is modified. Click on the **Execute** button to execute the statements.
	
	```
	PushTopic pushTopic = new PushTopic();

	pushTopic.Name = 'AccountObjectUpdates';

	pushTopic.Query = 'SELECT Id, AccountNumber, Name, BillingAddress, BillingStreet, BillingCity, BillingState,BillingPostalCode FROM Account';

	pushTopic.ApiVersion = 39.0;

	pushTopic.NotifyForOperationCreate = false;
	pushTopic.NotifyForOperationUpdate = true;
	pushTopic.NotifyForOperationDelete = false;
	pushTopic.NotifyForFields = 'All';

	insert pushTopic;
	```


6. Reopen the **Open Execute Anonymous Window**.  Clear all existing statements and enter the following **Apex** code to create a new Push Topic that will generate an event for every existing Salesforce Account object that is deleted. Click on the **Execute** button to execute the statements.

	```
	PushTopic pushTopic = new PushTopic();

	pushTopic.Name = 'AccountObjectDeletes';

	pushTopic.Query = 'SELECT Id, AccountNumber, Name, BillingAddress, BillingStreet, BillingCity, BillingState,BillingPostalCode FROM Account';

	pushTopic.ApiVersion = 39.0;

	pushTopic.NotifyForOperationCreate = false;
	pushTopic.NotifyForOperationUpdate = false;
	pushTopic.NotifyForOperationDelete = true;
	pushTopic.NotifyForFields = 'All';

	insert pushTopic;
	```

7. Close the **Force.com Developer Console** window.

## Step 3 Prepare the MQ Salesforce Bridge

The MQ Salesforce Bridge is the key component that supports integration between IBM MQ and Salesforce.  

* The Bridge subscribes to **Push Topic** events that are generated in Salesforce and then republishes these events as MQ Publish and Subscribe messages for other applications to subscribe to.
* The Bridge also subscribes to an IBM MQ topic of **/\<root\>/mqtosfb/event/+** (Where **\<root\>** is configured when setting up the Bridge), which will then forward messages published in that topic space to the Salesforce **Platform Event** infrastructure.  
  Complete the following steps to configure the MQ Salesforce Bridge. 
1. Open a Terminal window by right clicking on the desktop and clicking on the **Open Terminal Here** menu item.

	![](./images/pots/mq-sfdc/salesforce_open_terminal_window.png)

2. You may need to set the MQ environment variables whenever you open a new Terminal window.  Enter the following command to set the MQ environment.  
	
	{% include note.html content="Ensure that you include the preceding period and space before the start of the command." %}

	```
	. /opt/mqm/bin/setmqenv -n Installation1
	``` 
	![](./images/pots/mq-sfdc/salesforce_setmqenv_command.png)
	
	
3. Next you will need to create a keystore database to store the keystore file that was downloaded from Salesforce.  Enter the following command to launch the **IBM Key Manager** application.   
	
	```
	strmqikm
	```

	![](./images/pots/mq-sfdc/salesforce_strmqikm_command.png)
	
4. The **IBM Key Management** application will appear.  Click on the **Open** icon, select a **Key database type** of **JKS**, select the exported keystore that you copied to the **/home/student** directory and then click on the **OK** button.

	![](./images/pots/mq-sfdc/salesforce_open_keystore.png)

5. Enter **passw0rd** at the **Password** prompt and then click on the **OK** button.

	![](./images/pots/mq-sfdc/salesforce_keystore_password_prompt.png)

6. The contents of the keystore are now loaded into the **IBM Key Management** tool.  As of 2018 Salesforce uses DigiCert to sign their certificates.  Thus, you will need to add the root signer certificates to the keystore.  From a web browser enter the following URL to download the DigiCert root certificate:

	```
	https://www.digicert.com/CACerts/DigiCertSHA2SecureServerCA.crt
	```

	![](./images/pots/mq-sfdc/salesforce_root_cert_1.png)

7. Select the option to save the file and then click on the **OK** button.

	![](./images/pots/mq-sfdc/salesforce_root_cert_2.png)

8. Return to the **IBM Key Manager** window and then click on the drop down menu and then select **Signer Certificates**.

	![](./images/pots/mq-sfdc/salesforce_signer_certificates.png)
	
9. Click on the **Add...** button and then select the root signer certifcate file that you had just downloaded, then click on the **OK** button.

	![](./images/pots/mq-sfdc/salesforce_root_cert_3.png)
	
10. Add an appropriate label for the root certificate and then click on the **OK** button.

	![](./images/pots/mq-sfdc/salesforce_root_cert_4.png)
	
11. Save the keystore and exit the **IBM Key Management** application.
12. You are now ready to create and configure the MQ Salesforce Bridge.  Although you can configure the Bridge to work with an existing queue manager, for this lab exercise you will create a new queue manager to work with the Bridge.  (The Bridge runs as an MQ service, so a restart of the queue manager will eventually be required to complete the configuration.)  Return to the Terminal window and enter the following commands to create and start a queue manager: 

	```
	crtmqm -p 1414 -u SYSTEM.DEAD.LETTER.QUEUE SQM1
	strmqm SQM1
	```
		
	![](./images/pots/mq-sfdc/salesforce_create_qm.png)

13. The **student** userid is a member of the **mqm** group.  New queue managers created on version 8 and later MQ installations include several new security features that restrict access to members of the **mqm** group when utilizing client connections.  Since configuring security is outside the scope of this PoT you will need to enter the following commands to reduce the number of steps required to complete this PoT.  First, open the MQ command shell for the **SQM1** queue manager:

	```
	runmqsc SQM1
	```
	Next run the following **mqsc** commands:
	
	```
	ALTER QMGR CHLAUTH(DISABLED)
	ALTER AUTHINFO(SYSTEM.DEFAULT.AUTHINFO.IDPWOS) AUTHTYPE(IDPWOS) CHCKCLNT(OPTIONAL)
	REFRESH SECURITY TYPE(CONNAUTH)
	END
	```
	![](./images/pots/mq-sfdc/salesforce_runmqsc_security.png)
	  
14. Now you will need to create several local queues that the Bridge requires. Enter the following command in the Terminal window:

	```
	cat /opt/mqm/mqsf/samp/mqsfbSyncQ.mqsc | runmqsc SQM1
	```
	![](./images/pots/mq-sfdc/salesforce_create_queue.png)

15. You will now need to enter a command that will provide a series of prompts to enable you to configure the Bridge.  Note that the flags provided with the command allow you to specify both an input file and an output file.  You would enter both flags if you are modifying an existing configuration.  Assuming that you have not previously configured the Bridge you should only use the **-o** flag to initially configure the MQ Salesforce Bridge: 

	
	Flag  | Purpose
	-----:|:--------------
	-f    | OPTIONAL:  The name of an existing configuration file to read.  This may be specified when you are modifying an existing Bridge configuration.
	-o    | The name of the configuration file used to store the settings. 
	
	```
	runmqsfb -o /home/student/sfb_config.cfg
	```
	![](./images/pots/mq-sfdc/salesforce_runmqsfb_command.png)

16. Use the following table to complete each prompt:

	Prompt                                    | Value to Enter
	:-----------------------------------------|:-------------------
	Queue Manager or JNDI CF                  | **SQM1**
	MQ Base Topic                             | **/sf**
	MQ Channel                                | **SYSTEM.DEF.SVRCONN**
	MQ Conname                                | **localhost(1414)**
	MQ Publication Error Queue                | *\<leave default\>*
	MQ CCDT URL                               | *\<leave default\>*
	JNDI implementation class                 | *\<leave default\>*
	JNDI provider URL                         | *\<leave default\>*
	MQ Userid                                 | *\<leave default\>*
	MQ Password                               | *\<leave default\>*
	Salesforce Userid (reqd)                  | ***\< Enter your Salesforce Username \>***
	Salesforce Password (reqd)                | ***\< Enter your Salesforce Password \>***
	Security Token                            | ***\< Enter your Salesforce Security Token \>***
	Login Endpoint                            | *\<leave default\>*
	Consumer Key                              | *\<leave default\>*
	Consumer Secret                           | *\<leave default\>*
	Personal keystore for TLS certificates    | ***\<Enter full path to keystore\>***
	Keystore password                         | **passw0rd** (Where **0** is numeral zero)
	Trusted store for signer certificates     | *\<leave default\>*
	Trusted store password                    | *\<leave default\>*
	Use TLS for MQ connection                 | *\<leave default\>*
	PushTopic Names Add new topics            | **AccountObjectCreates, AccountObjectUpdates, AccountObjectDeletes** *\<Entered as a single line\>*
	Platform Event Names                      | *\<leave default\>*
	MQ Monitoring Frequency                   | *\<leave default\>*
	At-least-once delivery? (Y/N)             | *\<leave default\>*
	Subscribe to MQ publications for platform events? (Y/N) | **Y**
	Publish control data with the payload?    | *\<leave default\>*
	Delay before starting to process events   | *\<leave default\>*
	Runtime logfile for copy of stdout/stderr | **/home/student/mqsfbridge.log**


17. Change the file permissions on the newly created configuration file by entering the following command in the Terminal window:

	```
	chmod 766 /home/student/sfb_config.cfg
	```
	
	![](./images/pots/mq-sfdc/salesforce_chmod_config_file.png)

	
18. Now you must create an MQ Service on your queue manager that will automatically start the Bridge whenever the queue manager is started.  There is a template file that you can use to simplify the creation of the Service.  You will need copy and then tailor this file to suit your installation.  Use the following commands to copy and then edit the file:

	```
	cp /opt/mqm/mqsf/samp/mqsfbService.mqsc /home/student
	chmod +w /home/student/mqsfbService.mqsc
	gedit /home/student/mqsfbService.mqsc
	```

	{% include note.html content="You may receive an error message stating the **gedit** is not installed.  If so, follow the directions from the message to install it and then continue with this step." %}
	
	
	![](./images/pots/mq-sfdc/salesforce_gedit_file.png)
	

19. Edit the file as indicated in the following screen shot:

	Change the following line:
	
	```
	STARTARG('-m +QMNAME+ -f +MQ_INSTALL_PATH+/mqsf/samp/mqsfb.cfg')  +
	```
	to:
	
	```
	STARTARG('-m +QMNAME+ -f /home/student/sfb_config.cfg')  +
	```
	
	
	![](./images/pots/mq-sfdc/salesforce_edit_mqservice_template.png)


20. Save the file and exit the editor.
21. Enter the following command to configure the queue manager for the Bridge service:

	```
	cat /home/student/mqsfbService.mqsc | runmqsc SQM1
	```

22. Restart the queue manager to ensure that all configuration changes are active.

	```
	endmqm -i SQM1
	strmqm SQM1
	```
	
At this point your environment is now ready to start processing messages between your queue manager and your Salesforce instance.  Continue with the next step to setup static subscriptions to the Salesforce events and test out the Bridge's functions.



## Step 4 Test Your Work

To test out the IBM MQ Salesforce Bridge you will need to configure MQ subscriptions that will enable you to capture **Push Topic** event messages that are sent from Salesforce.  You will also need to publish a test message to your queue manager in order to see changes reflected in your Salesforce instance.  In this step of the PoT you will create a set of static Subscriptions that will capture create, update and delete actions against Account objects.

From this point forward we will use the IBM MQ Explorer to configure the static Subscriptions and test out Bridge operations.

1. Launch the IBM MQ Explorer by double-clicking on the **MQ Explorer** icon the desktop.  

	![](./images/pots/mq-sfdc/salesforce_launch_mq_explorer.png)
 
2. Expand the tree structure to expose the **Queues** folder of the **SQM1** queue manager.  Then right click on the **Queues** folder and select the **New** and **Local Queue...** menu items. The **Create a Local Queue** dialog will appear.

	![](./images/pots/mq-sfdc/salesforce_create_queue_menu.png) 
	
3. You will need to create a total of 4 queues for your static Subscriptions to route messages to.  Use the **Create a Local Queue** dialog to create the following queues:

	Queue Name            | Purpose
	:---------------------|:-------------------------
	ACCOUNTOBJECT_CREATES | Receive events related to creation of a new Account object.  
	ACCOUNTOBJECT_DELETES | Receive events related to the deletion of an Account object.
	ACCOUNTOBJECT_UPDATES | Receive events related to the update of an Account object.
	SALESFORCE_EVENTS     | Receives all events related to Account objects.  
	
	![](./images/pots/mq-sfdc/salesforce_create_local_queue.png) 

4. You will now need to create the static Subscriptions that will route the Salesforce events captured from the Bridge and send them to the queues you had just created.  Right click on the **Subscriptions** folder and then click on the **New** and **Subscription** menu items.  The **New Subscription** dialog will appear.

	![](./images/pots/mq-sfdc/salesforce_create_subscription_menu.png)

5. From the **New Subscription** dialog you will first need to name the static Subscription.  For this lab exercise use the same name as the queues that you had created.  

	![](./images/pots/mq-sfdc/salesforce_create_subscription_1.png)

	{% include note.html content="This is not a requirement.  It is simply a convenient naming convention." %}
	
	Refer to the following table for the proper entries to use for each static Subscription:
	
	Name                  | Topic String                   | Destination Name
	:---------------------|:-------------------------------|:--------------------------
	ACCOUNTOBJECT_CREATES | /sf/topic/AccountObjectCreates | ACCOUNTOBJECT_CREATES
	ACCOUNTOBJECT_DELETES | /sf/topic/AccountObjectDeletes | ACCOUNTOBJECT_DELETES
	ACCOUNTOBJECT_UPDATES | /sf/topic/AccountObjectUpdates | ACCOUNTOBJECT_UPDATES
	SALESFORCE_EVENTS     | /sf/#                          | SALESFORCE_EVENTS
	

6. The next page of the dialog is where you specify the Topic string that you are subscribing to as well as the name of the queue (**Destination name:**) that you want the subscription to route the messages to.  Click on the **Finish** button when ready.

	![](./images/pots/mq-sfdc/salesforce_create_subscription_2.png)


7. Once you have created all four subscriptions you are ready to test your work.  Return to the Salesforce web page and navigate to the list of all Accounts.  

	![](./images/pots/mq-sfdc/salesforce_navigate_all_salesforce_accounts.png)

8. You will first test creating a new account.  Click on the **New Account** button.

	![](./images/pots/mq-sfdc/salesforce_create_new_account.png)

9. Fill in the **Account Name** and the **Account Number** fields and then click on the **Save** button.  A new Salesforce Account object will be created.

	![](./images/pots/mq-sfdc/salesforce_new_account_data.png)

10. Return to the MQ Explorer and click on the **Queues** folder.  Observe that there is an entry in both the **ACCOUNTOBJECT_CREATES** and the **SALESFORCE_EVENTS** queues.  

	{% include note.html content="Note that the MQ Explorer refreshes its contents approximately every 20 seconds.  You may click on the **Refresh** button if you would like to immediately refresh the list contents." %}
	
	![](./images/pots/mq-sfdc/salesforce_mq_objects_creates.png)

11. Return to the Salesforce page and then edit the Account object that you had just created to include additional data, such as the website of your employer.  

	![](./images/pots/mq-sfdc/salesforce_mq_objects_updates.png)

12. Again, check the MQ Explorer and observe that there is now an entry in the **ACCOUNTOBJECT_UPDATES** queue as well as an additional entry in the **SALESFORCE_EVENTS** queue.

	![](./images/pots/mq-sfdc/salesforce_mq_explorer_object_updates.png)

13. Continue exploring how actions taken against Salesforce Account objects will be reflected in the MQ queues that have been configured in this lab.  You may also want to use the MQ Explorer or command line tools to review the content of the messages that have been sent from Salesforce. 

	{% include note.html content="You may also want to review the Salesforce [Push Topics](#pushtopics) section above to see how the various data fields that were included in the message payload was determined." %}
	
14. Now that you have had an opportunity to see how changes to Salesforce objects can be published to an IBM MQ queue manager, you will test how the Salesforce Bridge can forward messages published to the queue manage to Salesforce as a **Platform Event**.  Start by selecting an existing Salesforce Account and noting both the **Account Name** and the **Rating** values.

	![](./images/pots/mq-sfdc/salesforce_sample_account.png)

	{% include note.html content="Note that Account objects in Salesforce are customizable.  The two fields that have been selected are standard fields for all Account objects." %}

15. You will need to format a JSON message that will ultimately be consumed by the Salesforce Trigger that was created earlier.  The format of the message will need to match the format expected by the **Update\_Account\_Rating** Platform Event. Note that for this lab you will need to substitute the field value for the **Account\_Name\_\_c** field with the value found in the Account object that you selected.  The field value for the **Rating\_\_c** field may be one of **Hot**, **Warm** or **Cold**.  Select one of the values that will enable you to observe updates to the Salesforce Account object that you have selected. 

	![](./images/pots/mq-sfdc/salesforce_sample_JSON.png)

16. Next you will need to determine the proper Topic String to publish messages to.  

	![](./images/pots/mq-sfdc/salesforce_topic_space.png)

	Which for this lab will be:

	```
	/sf/mqtosfb/event/Update_Account_Rating__e
	```

17. A convenient way to publish a message on a Topic is to use the MQ Explorer.  Right click on the **Topics** folder in the MQ Explorer and then click on the **Test Publication...** menu item.

	![](./images/pots/mq-sfdc/salesforce_start_topic.png)

18. Enter your values in the **Topic String:** and **Message Data:** fields and then click on the **Publish message** button.

	![](./images/pots/mq-sfdc/salesforce_publish_test_message.png)

19. Verify that the updated value is shown in Salesforce.

	![](./images/pots/mq-sfdc/salesforce_updated_account.png)


## Summary

**Congratulations!** You have successfully completed this PoT.

If the Salesforce account that you used for this exercise is needed for other business purposes you may want to delete the Push Topics that were created as a part of this PoT.  Refer to the steps that were outlined in the Salesforce [Push Topics](#pushtopics) section above and use the following commands to remove the Push Topics that you created.

```
List<PushTopic> pts = [SELECT Id FROM PushTopic WHERE Name = 'AccountObjectCreates'];
pts.add([SELECT Id FROM PushTopic WHERE Name = 'AccountObjectUpdates']);
pts.add([SELECT Id FROM PushTopic WHERE Name = 'AccountObjectDeletes']);
Database.delete(pts);
```
