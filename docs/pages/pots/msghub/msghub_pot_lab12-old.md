---
title: Kafka Source Connector for IBM MQ
toc: false
sidebar: labs_sidebar
folder: pots/msghub
permalink: /msghub_pot_lab12.html
summary: Connect MQ to Event Streams
applies_to: [developer,administrator]
---

# Kafka source connector for IBM MQ
In this lab exercise you will connect IBM MQ to IBM Event Streams using the Kafka source connector for MQ. 

## Introduction

You can use the Kafka Connect source connector for IBM MQ to copy data
from IBM MQ into IBM Event Streams or Apache Kafka. The connector copies
messages from a source MQ queue to a target Kafka topic. There is also
an MQ sink connector that takes messages from a Kafka topic and
transfers them to an MQ queue. Running it is very similar to the source
connector. In this lab we will only be covering the source connector.

To begin with we will install the MQ on to ICP for our MQ messages. We
will then configure the source connector to run some tests to a local
stand-alone worker. We will then adjust our configuration to send the
messages to a topic in Event Streams and run a console consumer to
consume the messages.

At the end of this lab you should be able to install and configure MQ on
ICP, configure and test sending messages from MQ to Event Streams.

### What is Kafka Connect? 

Kafka Connect is a framework for connecting Kafka to external systems. It provides a standard way of writing and running connectors.

Use Kafka Connect to reliably move large amounts of data between your Kafka cluster and external systems. For example, it can ingest data from sources such as databases and make the data available for stream processing. 

![](./images/pots/msghub/lab12/image1.png)

### Source and sink connectors

Kafka Connect has two kinds of connectors: source connectors to import data from external systems, and sink connectors to export data to external systems.

A wide range of connectors exist, created by individuals and organisations. In addition, you can write your own connectors.

![](./images/pots/msghub/lab12/image2.png)

### Workers

Kafka Connect connectors run inside Java processes called workers. Kafka Connect can run in either standalone or distributed mode.

Standalone mode is intended for testing and temporary connections between systems, and all work is performed in a single process. Distributed mode is more appropriate for production use, as it benefits from additional features such as automatic balancing of work, dynamic scaling up or down, and fault tolerance.

![](./images/pots/msghub/lab12/image3.png)

### Connector catalog

The Event Streams Connector catalog contains a list of tried and tested connectors from both the community and IBM.

Support for community connectors is provided by the people that created them. If you have support for Event Streams, this includes support for IBM supported connectors.

Launch Connector catalog

![](./images/pots/msghub/lab12/image4.png)

### Connectors for IBM MQ

Connectors are available for copying data between IBM MQ and Event Streams. There is a MQ source connector for copying data from IBM MQ into Event Streams or Apache Kafka, and a MQ sink connector for copying data from Event Streams or Apache Kafka into IBM MQ.

For more information about MQ connectors, see the topic about connecting to IBM MQ.

### Setting up connectors

Event Streams provides help with setting up your Kafka Connect environment, adding connectors to that environment, and starting the connectors. See the instructions about setting up and running connectors.

### Running connectors on IBM Cloud Pak for Integration

If you have IBM MQ or another service running on IBM Cloud Private, you can use Kafka Connect and one or more connectors to flow data between your instance of IBM Event Streams and the service on IBM Cloud Private. In this scenario it makes sense to run Kafka Connect in IBM Cloud Private as well. See the instructions about running Kafka Connect and connectors on IBM Cloud Private


## Setting up and running connectors

IBM Event Streams helps you set up a Kafka Connect environment, prepare the connection to other systems by adding connectors to the environment, and start Kafka Connect with the connectors to help integrate external systems.

### Setting up a Kafka Connect environment

Set up the environment for hosting Kafka Connect. You can then use Kafka Connect to stream data between Event Streams and other systems.

Kafka Connect can be run in standalone or distributed mode. For more details see the explanation of Kafka Connect workers. Kafka Connect includes shell and bash scripts for starting workers that take configuration files as arguments.

For best results running Kafka Connect alongside Event Streams start Kafka Connect in distributed mode in Docker containers. In this mode, work balancing is automatic, scaling is dynamic, and tasks and data are fault-tolerant. To begin using Kafka Connect in distributed mode, follow the steps below, then add the connectors to your other systems and start Kafka Connect in its Docker container.
	
### Create topics

When running in distributed mode Kafka Connect uses three topics to store configuration, current offsets and status. In standalone mode Kafka Connect uses a local file.

Now that you are familiar with topics and creating them from the previous labs, you need to create a new topic for running the MQ source connector. 

1. In the Event Streams UI, click the *Topics* icon on the menu bar. 
	
	![](./images/pots/msghub/lab12/image6.png)

1. Click *Create topic*. Turn *Show all available options* to **On**. Use the following values when creating the topics. Create the following topics:

    ```
    mq2es - This topic will be the destination of the IBM MQ source connector
    mqtoevent - This topic will be the destination of the IBM MQ source connector
    ```
    
    Use the following values when creating the topics.

	| Topic Name | Purpose | Partitions |Replicas | Cleanup Policy |
	|:----------:|:-------:|:----------:|:-------:|:--------------:|
	| mq2es | Topic to receive messages from IBM MQ queue |  1 | 3 | compact | 
	| mqtoevent | Topic to receive messages from IBM MQ queue |  1 | 3 | compact |
		
	![](./images/pots/msghub/lab12/image7.png)
	
	{% include note.html content="When all available options is *On* you will see a great amount of detail for each topic. You only need to edit the *Core configuration* section. If you turn the switch to *Off* you will need to click *Next* three times before the final pane where you click **Create topic**." %}

1. Open a new browser tab and click the *Cloud Pak Navigator* bookmark. Click *View instances* then click mq-1 to open the MQ console. 

		
	

### Provide or generate an API Key

This API key must provide permission to produce and consume messages for all topics, and also to create topics. This will enable Kafka Connect to securely connect to your IBM Event Streams cluster.

1. Still in the *Event Streams UI*, click *Toolbox* in the primary navigation. Scroll to the *Connectors* section.

1. Go to the *Set up a Kafka Connect environment* tile, and click *Set up*.
    
1. Go to step 4 and click *Generate API Key*. This API key provides permission to produce and consume messages, and also to create topics. Wait for key to be generated. Copy and save the API Key in your *creds.txt* file.

	![](./images/pots/msghub/lab12/image8.png)

1. On step 5 click *Download Kafka Connect ZIP* to download the compressed file. Click *Save file* which will store it in your Downloads directory.

	![](./images/pots/msghub/lab12/image9.png)
	
1. Open a terminal and create a directory for *Kafak Connect* in your preferred location. Then abstract the downloaded kafkaconnect.zip. Use the following commands:

	```
	cd ~
	mkdir kafkaconnect
	cd kafkaconnect
	unzip ~/Downloads/kafkaconnect.zip
	``` 

	![](./images/pots/msghub/lab12/image10.png)
	
### Download Kafka Standalone 

1. Open a new tab in the Firefox browser and navigate to https://github.com/ibm-cloudintegration/mqmq-eventstream-standalone. 

	![](./images/pots/msghub/lab12/image33.png)
	
1. Click *kafka_standalone.zip*. 

	![](./images/pots/msghub/lab12/image34.png)

1. Click *Download*, the save the file. 

	![](./images/pots/msghub/lab12/image35.png)

1. In a terminal window, in your home directory, unzip the download with the following command: 

	```
	cd ~
	unzip ~/Downloads/kafka_standalone.zip
	```
		
	
		
### Adding connectors to your Kafka Connect environment

You will now prepare Kafka Connect for connections to other systems by adding the required connectors. 

To run a particular connector, Kafka Connect must have access to a JAR file or set of JAR files for the connector. The quickest way to do this is by adding the JAR file(s) to the classpath of Kafka Connect. This is not the recommended approach because it does not provide classpath isolation for different connectors.

Since version 0.11.0.0 of Kafka the recommended approach is to configure the plugin.path in the Kafka Connect properties file to point to the location of your connector JAR(s).

1. Return to the Event Streams browser. Click  *Next* at the bottom of the page. You can also access this page by clicking Toolbox in the primary navigation, scrolling to the *Connectors* section, and clicking *Add connectors* on the *Add connectors to your Kafka Connect environment* tile.

	![](./images/pots/msghub/lab12/image11.png)

1. Click the *Toolbox* breadcrumb at the top of the window. Scroll down to *Connector* and click *Connecting to IBM MQ?*.

	![](./images/pots/msghub/lab12/image18.png)

1. Under *Download the connector JAR and configuration*, make sure *MQ Source* is highlighted and click the *Download MQ Soure JAR* button. Then click *Save File*.

	![](./images/pots/msghub/lab12/image19.png)

1. The jar is stored in your Downloads directory. You must copy the jar to the *connectors* subdirectory of *kafkaconnect*. 

	{% include note.html content="The docker image will copy all connectors in the /connectors directory into the container on the plugin.path. You need to copy the connector JAR file(s) you want to have available into the /connectors directory." %}

1. Copy the MQ Source JAR file to /home/ibmuser/kafkaconnect/connectors. 

	```
	cp ~/Downloads/mqjar ~/kafkaconnect/connectors
	```
	
	![](./images/pots/msghub/lab12/image17.png)

1. Build Kafka Connect Docker container. In the terminal window, you should still be in the /home/ibmuser/kafkaconnect directory. 

1. Make Docker is running. Enter the following command:
	
	```
	sudo systemctl start docker
	```
	
	You will know that Docker is now running when you return to the command line with no output.

1. In the *kafkaconnect* directory review the contents especially the *Dockerfile* to see how the container will be built.

	![](./images/pots/msghub/lab12/image21.png)
	
1. Log in to docker with the following commmand:

	```
	sudo docker login -u cp -p eyJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJJQk0gTWFya2V0cGxhY2UiLCJpYXQiOjE1NzY1MTM2NTUsImp0aSI6IjIyZjdiZGFkYTUyMTQ2ZDRiYzE4ZWM0ZGVmNTk5Yjk2In0.IT8gBkn6TjtXvbaPS7VEOxSVeK1GLxsbBn0c5t-5SlA cp.icr.io
	```
	
	![](./images/pots/msghub/lab12/image25.png)

1. Build the *Kafka Connect* Docker image with the command:

	```
	sudo docker build -t kafkaconnect:0.0.1 .
	```

	![](./images/pots/msghub/lab12/image22.png)

1. Run the Docker container with command:

	```
	docker run -v $(pwd)/config:/opt/kafka/config -p 8083:8083 kafkaconnect:0.0.1
	```
	
	![](./images/pots/msghub/lab12/image23.png)
	
1. Verify that your chosen connectors are installed in your Kafka Connect environment by running a curl command:

	```
	curl http://localhost:8083/connector-plugins
	```
	
	A list of connector plugins available is displayed.	
	![](./images/pots/msghub/lab12/image24.png)

1. Now that the connector jar is available, you need to configure MQ. Return to the web browser and click *Download MQ Source configuration*. This will bring up a configuration properties panel. Use this table to enter the properties:


	| Property | Value | 
	|:----------:|:-------:|
	| Topic | eslab |   
	| Queue manager | mq |
	| Connection mode | client |
	| Queue | Q1 |
	| Connection name list | https://mq-1-ibm-mq-qm-mq.apps.demo.ibmdte.net/ |

	When complete click *Downlaod*, then *Save File*. 
	
	![](./images/pots/msghub/lab12/image26.png)

1. Click the *X* to close the configuration window

### Part 2 - Connector

We now move on to installing and testing the connector.

#### 2.1 - Get the connector files

1. Go to the Events Streams UI and select the Toolbox.

1. Scroll down to the Connectors tile.

1. Click on "Find out more".

	![](./images/pots/msghub/lab6/image21.png)

1. On the right hand side you will see downloads for both the Connector JAR and a Sample connector properties file.

1. Download these files and copy them to the */home/student directory*. 

	![](./images/pots/msghub/lab6/image22.png) 
	
	{% include note.html content="The connector is also available in GitHub if you wish to build it yourself.
[[https://github.com/ibm-messaging/kafka-connect-mq-sink](https://github.com/ibm-messaging/kafka-connect-mq-sink)" %}

Kafka Connect connectors run inside a Java process called a *worker*.
Kafka Connect can run in either standalone or distributed mode.
Standalone mode is intended for testing and temporary connections
between systems. Distributed mode is more appropriate for production
use. These instructions focus on standalone mode because it\'s easier to
see what\'s going on.

When you run Kafka Connect with a standalone worker, there are two
configuration files. The worker configuration file contains the
properties needed to connect to Kafka. The connector configuration file
contains the properties needed for the connector. So, configuration to
connect to Kafka goes into the worker configuration file, while the MQ
configuration goes into the connector configuration file.

It\'s simplest to run just one connector in each standalone worker.
Kafka Connect workers throw out a lot of messages and it\'s much simpler
to read them if the messages from multiple connectors are not
interleaved.

We are going to start from scratch with a local Kafka installation
before moving on to Event Streams.

#### 2.2 - Install local Kafka


1. Go to the Apache Kafka download site.

	<http://kafka.apache.org/downloads>

1. Select the .tgz file (called something like kafka\_2.11-2.0.0.tgz)
    and unpack it (to your home directory). Kafka is continuously updated so the releases available will probably be different than what is shown in the screen shot. 

	![](./images/pots/msghub/lab6/image23.png)

1. You are given options for download mirror sites. Click the suggested site to download the .tgz file. 

	![](./images/pots/msghub/lab6/image44.png)

1. In your terminal window, make sure you are in the home directory and unpack the tar file (to your home directory) with the following command.

	```
	tar -zxvf ~/Downloads/kafka_2.11-2.0.0.tgz
	```
	![](./images/pots/msghub/lab6/image45.png)
	
	The top-level directory of the unpacked .tgz file is referred to as the
Kafka root directory. It contains several directories including bin for
the Kafka executables and config for the configuration files. 

	There are several components required to run a minimal Kafka cluster.
It\'s easiest to run them each in a separate terminal window, starting
in the Kafka root directory.

1. You now need to update the connector config file
    (mq-source.properties) for the value required to connect to your
    queue manager.
    
    ![](./images/pots/msghub/lab6/image46.png)

1. Open this file in a text editor and complete the values as follows:

	a.  mq.queue.manager=QM1
	
	b.  mq.connection.name.list=10.0.0.1(your mq listener NodePort)
	
	c.  mq.channel.name=DEV.APP.SVRCONN
	
	d.  mq.queue=DEV.QUEUE.1
	
	e.  topic=eslab
	
1. Click *Save* to save the file and then close it.
	
	{% include note.html content="We could use a different topic but when we come to the Event Streams testing, we already have a producer and a consumer Service ID from previous labs, so we will reuse them to save a little time and effort." %} 
	
	![](./images/pots/msghub/lab6/image24.png) 

We are now ready to start some testing.

### Part 3 - Test Connector from MQ to local Kafka installation

1. Open a new terminal window.

1. Change to the kafka root directory

1. Start a ZooKeeper server:

	```
	bin/zookeeper-server-start.sh config/zookeeper.properties
	```
	
	![](./images/pots/msghub/lab6/image25.png)

1. Wait while it starts up and then prints a message similar to this:

	![](./images/pots/msghub/lab6/image26.png)

1. Open another terminal.

1. Again, change to the kafka root directory

1. Start a Kafka server:

	```
	bin/kafka-server-start.sh config/server.properties
	```
	
	![](./images/pots/msghub/lab6/image27.png)
	
1. Wait while it starts up and then prints a message similar to this:

	"INFO \[KafkaServer id=0\] started"

	We now need to create the topic we will use.

1. Open a terminal window (did we mention that there are lots of
    windows for this...)
   
1.  Change to the kafka root directory (did we mention this before...)

1. Enter the following command:

	```
	bin/kafka-topics.sh --zookeeper localhost:2181 --create --topic eslab --partitions 1 --replication-factor 1
	```

1. Check that the topic was created successfully: 

	```
	bin/kafka-topics.sh --zookeeper localhost:2181 --describe
	```

	
	You now have a Kafka cluster consisting of a single node. This
configuration is just a toy, but it works fine for a little testing.

	The configuration is as follows:

	* Kafka bootstrap server - localhost:9092
	
	* ZooKeeper server - localhost:2181
	
	* Topic name -- eslab

	{% include note.html content="This configuration of Kafka puts its data in /tmp/kafka-logs, while ZooKeeper uses /tmp/zookeeper and Kafka Connect uses /tmp/connect.offsets. You can clear out these directories to reset to an empty state, making sure beforehand that they\'re not being used for something else." %}	

	We are now ready to start the connector.

1. Enter the following command:

	```
	CLASSPATH=/home/student/kafka-connect-mq-source-1.0.1-jar-with-dependencies.jar bin/connect-standalone.sh config/connect-standalone.properties ~/mq-source.properties
	```

	![](./images/pots/msghub/lab6/image28.png)

1. If the connector is running correctly you will see two messages.
    The first indicates that the connector has successfully connected to
    the queue manager. The second will indicate that the connector is
    now polling for messages.

	Last, but not least we need to start the console consumer to consume the
messages that will be publish from MQ to the eslab topic.

1. Open (yet) another terminal window and enter the following command:

	```
	cd kafka_2.11-2.1.1
	bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic eslab
	```

	![](./images/pots/msghub/lab6/image29.png)

	You will not see anything until it starts consuming messages.

1. Go back to the MQ console.

1. Select the DEV.QUEUE.1 (or whichever queue you used in your
    configuration file).

1. Put a sample message on the queue.

	![](./images/pots/msghub/lab6/image31.png)

	![](./images/pots/msghub/lab6/image32.png)

1. You should see this message in the console consumer.

	![](./images/pots/msghub/lab6/image33.png)

1. Woohoo! You have successfully have your local Kafka consume the
    message that has been published from MQ.

1. You can now close the console consumer (ctrl-C).

1. You will see the number of messages that have been consumed).

	![](./images/pots/msghub/lab6/image34.png)

1. After you have finished experimenting with this, you will probably
    want to stop Apache Kafka. You start by stopping any Kafka Connect
    workers (ctrl-C).

1. Then, in the Kafka root directory, stop Kafka:

	```
	bin/kafka-server-stop.sh
	```

	![](./images/pots/msghub/lab6/image35.png)

1. And finally, stop ZooKeeper:

	```
	bin/zookeeper-server-stop.sh
	```

	![](./images/pots/msghub/lab6/image36.png)

1. You can close all those extra windows now.

### Part 4 - Test Connector from MQ to Event Streams

If you have not consumed all the messages from the load test -- you
might want to go and run the consumer test app and get rid of them all
before you proceed (remember that we produced A LOT of messages). Grab a
cuppa while this happens.

You can use an existing MQ or Kafka installation, either locally or on
the cloud. For performance reasons, it is recommended to run the Kafka
Connect worker close to the queue manager to minimise the effect of
network latency. So, if you have a queue manager in your datacenter and
Kafka in the cloud, it\'s best to run the Kafka Connect worker in your
datacenter.

To use an existing Kafka cluster, you specify the connection information
in the worker configuration file.

You will need:

* A list of one or more servers for bootstrapping connections
	
* Whether the cluster requires connections to use SSL/TLS
	
* Authentication credentials if the cluster requires clients to
	authenticate
	
You will also need to run the Kafka Connect worker (which we ran in the
previous section).

Start by creating a new api key for a producer and a consumer
    
1. In the IBM Event Streams UI, click *Connect to this cluster* or 
*Connect to this topic*.
    
    ![](./images/pots/msghub/lab6/image36a.png) 
    
1. In the API key section enter *producer* as the name of your application 
and click *Produce only*.
    
    ![](./images/pots/msghub/lab6/image36b.png)
    
1. Enter the topic name *eslab* and click *Generate API key*

	![](./images/pots/msghub/lab6/image36c.png)   
	
1. The API key is created. You need to download the json file. Notice that the file is always named **es-api-key.json**. Once downloaded, you rename it to **producer.json**.
	
	![](./images/pots/msghub/lab6/image36d.png)

1. Download and save the Java truststore.

	![](./images/pots/msghub/lab6/image36e.png)
	
1. Repeat the above steps instead choosing *Consume only*. Give the application name as *consumer and dame the downloaded file **consumer.json**.

1. We will be using them as follows (that's why it is a good idea to rename them appropriately).

	a.  Kafka connect = producer
	
	b.  Console consumer = consumer

1. Go to the config directory in the kafka
    (/home/student/kafka\_2.11-2.0.0/config)

1. Take a copy of the connect-standalone.properties file
    (connect-standalone.properties). We named ours
    **connect-standalone-es.properties.**

1. Edit the properties file as follows using the following command:
	
	```
	gedit connect-standalone-es.properties
	```
	
	a.  The bootstrap server = the bootstrap server listed in the Event Streams UI in the connection panel where you created the API keys.
	
	b.  The truststore location = /home/student/Downloads/es-cert.jks
	
	c.  The truststore password = password
	
	d.  The password for the sasl.jaas.config is the API key. Copy the key from the producer.json file.

	{% include warning.html content="REMEMBER THIS IS THE PRODUCER SO USE THE RIGHT KEY." %}

	![](./images/pots/msghub/lab6/image37e.png)
	
	```
	security.protocol=SASL_SSL
	ssl.protocol=TLSv1.2
	ssl.endpoint.identification.algorithm=
	ssl.truststore.location=/home/student/Downloads/es-cert.jks
	ssl.truststore.password=password
	sasl.mechanism=PLAIN
	sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="token" password="";
	
	producer.security.protocol=SASL_SSL
	producer.ssl.protocol=TLSv1.2
	producer.ssl.endpoint.identification.algorithm=
	producer.ssl.truststore.location=/home/student/Downloads/es-cert.jks
	producer.ssl.truststore.password=password
	producer.sasl.mechanism=PLAIN
	producer.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="token" password="";
	```

1. The MQ properties have not changed so there is nothing required for
    that file.

1. Start the connector ensuring that you point at the new configuration
    file.
	
	```
	CLASSPATH=/home/student/kafka-connect-mq-source-1.0.1-jar-with-dependencies.jar bin/connect-standalone.sh config/connect-standalone-es.properties ~/mq-source.properties
	```

	![](./images/pots/msghub/lab6/image38.png)

1. Check for the two messages as before to indicate the you are
    connected to the queue mamager and that the connector is polling.

	![](./images/pots/msghub/lab6/image39.png)

1. Go back to the MQ console and put a message on the queue again.

	![](./images/pots/msghub/lab6/image42.png)

1. Return to your Event Streams UI and click on your topic in the *Topics* tab.

1. Under *Messages* you should see the message from MQ.

	![](./images/pots/msghub/lab6/image37d.png)
	
	Did it work? Well done!
	
### Part 5 - Test Connector from MQ to Event Streams Using a console connector

There is another way to test it out. You can start a console consumer.

73. Take a copy of the *connect-standalone-es.properties* and save it to your home directory as *mqlab.properties*.

	![](./images/pots/msghub/lab6/image39a.png)

1. Change to your home directory and edit the mqlab.properties file as follows using the following command:
	
	```
	gedit mqlab.properties
	```

1. Edit this file as shown below. The only property you need to change is the API key in the password. You can copy the snippet below, then add the consumer's key in the password fields (copy key from the consumer.json file).

	{% include warning.html content="REMEMBER THIS MUST BE THE CONSUMER APIKEY." %}

	![](./images/pots/msghub/lab6/image47.png)
	
	```
	security.protocol=SASL_SSL
	ssl.protocol=TLSv1.2
	ssl.endpoint.identification.algorithm=
	ssl.truststore.location=/home/student/es-cert.jks
	ssl.truststore.password=password
	sasl.mechanism=PLAIN
	sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="token" password="";
	
	consumer.security.protocol=SASL_SSL
	consumer.ssl.protocol=TLSv1.2
	consumer.ssl.endpoint.identification.algorithm=
	consumer.ssl.truststore.location=/home/student/es-cert.jks
	consumer.ssl.truststore.password=password
	consumer.sasl.mechanism=PLAIN
	consumer.sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="token" password="";
	```	
	
75. Enter the command for the console consumer. 

	```
	bin/kafka-console-consumer.sh --bootstrap-server 10.0.0.1:30151 --consumer.config /home/student/mqlab.properties --topic eslab --group eslabtester
	```

	![](./images/pots/msghub/lab6/image41a.png)
	
1. Return to the MQ console and put another message on the *DEV.QUEUE.1* queue. You should see it on your console consumer. 

	![](./images/pots/msghub/lab6/image48.png)
	
	Group should be the name of your application that you use for testing your topic, so if you are not using the name (eslab / eslabtester) in this guide then yours will be different.

Did it work? Well done, this completes the MQ connector lab.

For more information on using the MQ connector (both source and sink) go
to the help documentation for Event Streams or GitHub.

[https://github.com/ibm-messaging/kafka-connect-mq-
source/blob/master/UsingMQwithKafkaConnect.md](https://github.com/ibm-messaging/kafka-connect-mq-%20source/blob/master/UsingMQwithKafkaConnect.md)

## Congratulations
You have successfully created a workload generator application and ran some loads through Event Streams. 

[Continue to Lab 7 - Monitoring Event Streams](msghub_pot_lab7.html)