---
title: IBM Event Streams Toolbox - IBM Cloud Private
toc: false
sidebar: labs_sidebar
folder: pots/msghub
permalink: /msghub_pot_lab5.html
summary: Event Streams Workload Application 
applies_to: [developer,administrator]
---

# Producing Load on Event Streams with Workload Application

IBM Event Streams provides a high-throughput producer application you
can use as a workload generator to test message loads and help validate
the performance capabilities of your cluster.

You can use one of the predefined load sizes, or you can specify your
own settings to test throughput.

## Introduction

In this lab we will install the sample application and test running some
load.

At the end of this lab your should be able to install and configure the
sample application and run load tests.

1.  Log on to the master node as shown previously.

2.  Go to Event Streams UI as shown previously.

3.  Go to Toolbox.

	![](./images/pots/msghub/lab5/image1.png)

4.  Click on View in GitHub on the Workload Generator Application.

1.  When you arrive at the github repo, scroll down to the README.md file section and click the hyperlink under *Getting Started*.

	![](./images/pots/msghub/lab5/image23.png)
	
5.  Go to Releases.

    ![](./images/pots/msghub/lab5/image2.png)

6.  Download the sample application (es-producer.jar) under *Latest release*.

	![](./images/pots/msghub/lab5/image27.png)
	
	 If you get a warning about the download harming your computer, click *Keep*.
	
	![](./images/pots/msghub/lab5/image24.png)

8.  Open a command window.

7.  Move the downloaded file to your home directory..

	```
	mv ~/Downloads/es-producer.jar ~/
	```
	
	![](./images/pots/msghub/lab5/image26.png)

9.  Enter the following command to create the config file for the
    application:
	
	```
	java -jar es-producer.jar -g
	```
	
	![](./images/pots/msghub/lab5/image3.png)

10. The config file has been created (producer.config).

	![](./images/pots/msghub/lab5/image4.png)

11. Open this file in a text editor with the following commannd:

	```
	gedit producer.config
	```
	
	You will be prompted for student's password. Enter *Passw0rd!*.
	
	![](./images/pots/msghub/lab5/image4a.png)
	
1.  We now need to edit this file to add the following information:

	* bootstrap.servers

	* ssl.truststore.location

	* sasl.jacl.config.

	To get the required information we need to go back to the Topics view.
We will use the eslab topic that was created previously.

12. In the Topics view, click on the eslab topic to open it.

	![](./images/pots/msghub/lab5/image5.png)

13. Click on the *Connection to this topic* tab.

	![](./images/pots/msghub/lab5/image6.png)

14. Click on the Copy broker URL box to get the URL. (save this
    somewhere to be added to the config file later).

15. Scroll down a little until you see the Certificates section.

16. We are using a java application so click on the Download Java
    truststore.
    
    ![](./images/pots/msghub/lab5/image7.png)

17. Copy this file to your home directory for later use.

18. Under *API key*, enter a name for your application - **es-producer**. Click *Produce only*. 

	![](./images/pots/msghub/lab5/image8.png)

19. The topic is filled in for you. Click *Generate API key*.

	![](./images/pots/msghub/lab5/image9.png)

20. As you can see, the API key is generated. You should copy the *API key* and save it where you saved the bootstrap server value or download the key as a file (es-api-key.json). Copy this file to your home directory. 

	![](./images/pots/msghub/lab5/image10a.png) 
	
1.  Close the *Topic connection* pane by clicking the *X*. 

	![](./images/pots/msghub/lab5/image10b.png) 
	
1. We now need to go back to the text editor and update the configuration file with the correct information.

32. Enter the connection URL that you saved previously as the
    **bootstrap.servers** parameter
    
    ![](./images/pots/msghub/lab5/image13.png)

33. Enter the full path (including filename) of the keystore file you
    downloaded previously (note that in this example this file has been
    moved to the home directory).
    
    ![](./images/pots/msghub/lab5/image14.png)

34. Open the downloaded es-api-key.json file and copy the apikey.

35. Enter the API Key as the password parameter (do not change the
    username as this is already set to the required value of "token").
    
    ![](./images/pots/msghub/lab5/image15.png)

36. Save the file.

37. Go back to terminal window.

	You create a load on your IBM Event Streams Kafka cluster by running
the es-producer.jar command. You can specify the load size based on the
provided predefined values, or you can provide specific values for
throughput and total messages to determine a custom load.

	To use a predefined load size from the producer application, you use
the es-producer.jar with the -s option:	
	
	*java -jar target/es-producer.jar -t \<topic-name\> -s \<small/medium/large\>*
	

	We will start with a small load:

38. Enter command:
	
	```
	java -jar es-producer.jar -t eslab -s small
	```

39. After listing the parameters that will be used you will see the
    messages being sent (this should be fairly quick so if you are not
    seeing this, check for errors either on the screen or in the
    **producer.log** file).
    
    ![](./images/pots/msghub/lab5/image16.png)

40. The run will finish with a message that indicates the number of
    messages sent and the overall performance.
    
    ![](./images/pots/msghub/lab5/image17.png)

41. We can go back to the Event Streams UI and have a look at the
    Monitor for our test.

42. Click on the Monitor tab.

	![](./images/pots/msghub/lab5/image18.png)

43. We can see the incoming messages from our "small" test.

	![](./images/pots/msghub/lab5/image19.png)

44. There is not really very much interesting going on here (more about
    Monitoring will be contained in another lab). However, lets run a
    large load and we will be able to watch the monitor moving as the
    messages are loaded.

45. Go back to the terminal window and rerun the load test but this time
    send a large sized load.
    
    ```
    java -jar es-producer.jar -t eslab -s large
    ```

46. This is going to take considerably longer than before so go back to
    the Events Streams UI.

47. You will see that the volume of data is increasing.

48. Click on the **past 15 minutes** label for a more detailed view.

	![](./images/pots/msghub/lab5/image20.png)

49. We can see both the small load run from earlier and now the large
    load run in progress.
    
    ![](./images/pots/msghub/lab5/image21.png)

If you have time you may wish to experiment with options other than the
defaults. The details of all of the available parameters can be found
at:

[Parameters for Workload App](https://ibm.github.io/event-streams/getting-started/testing-loads)

If you still have your Consumer app running, you may also wish to start
consuming the messages and have a look at the Monitor.

## Congratulations
You have successfully created a workload generator application and ran some loads through Event Streams. You are now ready to create an Event Streams connector to MQ. 

[Continue to Lab 6 - Kafka Source Connector for MQ](msghub_pot_lab6.html)