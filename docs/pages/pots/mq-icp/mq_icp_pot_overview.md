---
title: IBM MQ Overview
toc: false
sidebar: labs_sidebar
folder: pots/mq-basic
permalink: /mq_basic_pot_overview.html
summary: Introduction to IBM MQ
applies_to: [developer administrator]
---

## Overview

This guide and associated VMware image helps you get started with IBM MQ on distributed platforms.

You can use IBM MQ to enable applications to communicate at different times and in many diverse computing environments.

What is IBM MQ?

* IBM MQ is messaging for applications. It sends messages across networks of diverse components. Your application connects to IBM MQ to send or receive a message. IBM MQ handles the different processors, operating systems, subsystems, and communication protocols it encounters in transferring the message. If a connection or a processor is temporarily unavailable, IBM MQ queues the message and forwards it when the connection is back online.
* An application developer has a choice of programming interfaces, and programming languages to connect to IBM MQ.
* IBM MQ is messaging and queuing middleware, with point-to-point,publish/subscribe, and file transfer modes of operation. Applications can publish messages to many subscribers over multicast.

You can use IBM MQ to enable applications to communicate at different times and in many diverse computing environments.

What is IBM MQ?

* IBM MQ is messaging for applications. It sends messages across networks of diverse components. Your application connects to IBM MQ to send or receive a message. IBM MQ handles the different processors, operating systems, subsystems, and communication protocols it encounters in transferring the message. If a connection or a processor is temporarily unavailable, IBM MQ queues the message and forwards it when the connection is back online.
* An application developer has a choice of programming interfaces, and programming languages to connect to IBM MQ.
* IBM MQ is messaging and queuing middleware, with point-to-point,publish/subscribe, and file transfer modes of operation. Applications can publish messages to many subscribers over multicast.

What is IBM MQ?

* IBM MQ is messaging for applications. It sends messages across networks of diverse components. Your application connects to IBM MQ to send or receive a message. IBM MQ handles the different processors, operating systems, subsystems, and communication protocols it encounters in transferring the message. If a connection or a processor is temporarily unavailable, IBM MQ queues the message and forwards it when the connection is back online.
* An application developer has a choice of programming interfaces, and programming languages to connect to IBM MQ.
* IBM MQ is messaging and queuing middleware, with point-to-point,publish/subscribe, and file transfer modes of operation. Applications can publish messages to many subscribers over multicast.

***Messaging***    
Programs communicate by sending each other data in messages rather than by calling each other directly.

***Queuing***    
Messages are placed on queues, so that programs can run independently of each other, at different speeds and times, in different locations, and without having a direct connection between them.

***Point-to-point***    
Applications send messages to a queue, or to a list of queues. The sender must know the name of the destination, but not where it is. 
   
***Publish/subscribe***    

Applications publish a message on a topic, such as the result of a game played by a team. IBM MQ sends copies of the message to applications that subscribe to the results topic. They receive the message with the results of games played by the team. The publisher does not know the names of subscribers, or where they are.

***Multicast***    
Multicast is an efficient form of publish/subscribe messaging that scales to many subscribers. It transfers the effort of sending a copy of a publication to each subscriber from IBM MQ to the network. Once a path for the publication is established between the publisher and subscriber, IBM MQ is not involved in forwarding the publication.

***File transfer***    
Files are transferred in messages. IBM MQ File Transfer Edition manages the transfer of files and the administration to set up automated transfers and log the results. You can integrate the file transfer with other file transfer systems, with IBM MQ messaging, and the web.

***Telemetry***   
MQ Telemetry is messaging for devices. IBM MQ connects device and application messaging together. It connects the internet, applications, services, and decision makers with networks of instrumented devices. IBM MQ Telemetry has an efficient messaging protocol that connects a large numbers of devices over a network. The messaging protocol is published, so that it can be incorporated into devices. You can also develop device programs with one of the published programming interfaces for the protocol.

**What can it do for me?**    
IBM MQ sends and receives data between your applications, and over networks.

Message delivery is assured and decoupled from the application. Assured, becauseIBM MQ exchanges messages transactionally, and decoupled, because applications do not have to check that messages they sent are delivered safely.

Applications publish a message on a topic, such as the result of a game played by a team. IBM MQ sends copies of the message to applications that subscribe to the results topic. They receive the message with the results of games played by the team. The publisher does not know the names of subscribers, or where they are.

***Multicast***    
Multicast is an efficient form of publish/subscribe messaging that scales to many subscribers. It transfers the effort of sending a copy of a publication to each subscriber from IBM MQ to the network. Once a path for the publication is established between the publisher and subscriber, IBM MQ is not involved in forwarding the publication.

***File transfer***    
Files are transferred in messages. IBM MQ File Transfer Edition manages the transfer of files and the administration to set up automated transfers and log the results. You can integrate the file transfer with other file transfer systems, with IBM MQ messaging, and the web.

***Telemetry***   
MQ Telemetry is messaging for devices. IBM MQ connects device and application messaging together. It connects the internet, applications, services, and decision makers with networks of instrumented devices. IBM MQ Telemetry has an efficient messaging protocol that connects a large numbers of devices over a network. The messaging protocol is published, so that it can be incorporated into devices. You can also develop device programs with one of the published programming interfaces for the protocol.

**What can it do for me?**    
IBM MQ sends and receives data between your applications, and over networks.

Message delivery is assured and decoupled from the application. Assured, becauseIBM MQ exchanges messages transactionally, and decoupled, because applications do not have to check that messages they sent are delivered safely.

You can secure message delivery between queue managers with SSL/TLS.
With Advanced Message Security (AMS), you can encrypt and sign messages between being put by one application and retrieved by another.
Application programmers do not need to have communications programming knowledge.

**How do I use it?**    

* Create and manage IBM MQ with the IBM MQ Explorer GUI, with the optional MQ Web Console, or by running commands from a command window or application.
* Program applications to send and receive messages by calling one of the programming interfaces. Programming interfaces are provided for different languages, and include the standard JMS programming interface, and classes for the Windows communication foundation.
* Send and receive IBM MQ messages from browsers with the HTTP protocol.
* Create and manage IBM MQ with the IBM MQ Explorer GUI, with the optional MQ Web Console, or by running commands from a command window or application.
* Program applications to send and receive messages by calling one of the programming interfaces. Programming interfaces are provided for different languages, and include the standard JMS programming interface, and classes for the Windows communication foundation.
* Send and receive IBM MQ messages from browsers with the HTTP protocol.
* Create and manage IBM MQ with the IBM MQ Explorer GUI, with the optional MQ Web Console, or by running commands from a command window or application.
* Program applications to send and receive messages by calling one of the programming interfaces. Programming interfaces are provided for different languages, and include the standard JMS programming interface, and classes for the Windows communication foundation.
* Send and receive IBM MQ messages from browsers with the HTTP protocol.
* Create and manage IBM MQ with the IBM MQ Explorer GUI, with the optional MQ Web Console, or by running commands from a command window or application.
* Program applications to send and receive messages by calling one of the programming interfaces. Programming interfaces are provided for different languages, and include the standard JMS programming interface, and classes for the Windows communication foundation.
* Send and receive IBM MQ messages from browsers with the HTTP protocol.

**How does it work?**

* An administrator creates and starts a queue manager with commands. Subsequently, the queue manager is usually started automatically when the operating system boots. Applications, and other queue managers can then connect to it to send and receive messages.
* An application or administrator creates a queue or a topic. Queues and topics are objects that are owned and stored by a queue manager.
* When your application wants to transfer data to another application, it puts the data into a message. It puts the message onto a queue, or publishes the message to a topic. There are three main ways that the message can be retrieved:
    * A point-to-point application connected to the same queue manager retrieves the message from the same queue.
    * For example, an application puts messages on a queue as way of storing temporary or persistent data. A second example: An application that shares data with another application that is running in a different process.
    * A point-to-point application connected to another queue manager retrieves the same message from a different queue.
    * Applications communicate with each other by exchanging messages on queues. The main use of IBM MQ is to send or exchange messages. One application puts a message on a queue on one computer, and another application gets the same message from another queue on a different computer. The queue managers on the two computers work together to transfer the message from the first queue to the second queue. The applications do not communicate with each other, the queue managers do.
    * A subscriber application connected to any queue manager retrieves messages on common topics.
    * A publisher application creates a message and publishes it to a topic on one computer. Any number of subscriber applications subscribe to the same topic on different computers. IBM MQ delivers the publication to queues that belong to the queue managers the subscribers are connected to. The subscribers retrieve the message from the queues.
* MQ channels connect one queue manager to another over a network. You can create MQ channels yourself, or a queue manager in a cluster of queue managers creates MQ channels when they are needed.
* You can have many queues and topics on one queue manager.
* You can have more than one queue manager on one computer.
* An application can run on the same computer as the queue manager, or on a different one. If it runs on the same computer, it is a IBM MQ server application. If it runs on a different computer, it is a IBM MQ client application. Whether it is IBM MQclient or server makes almost no difference to the application. You can build a client/server application with IBM MQ clients or servers.
* An application can run on the same computer as the queue manager, or on a different one. If it runs on the same computer, it is a IBM MQ server application. If it runs on a different computer, it is a IBM MQ client application. Whether it is IBM MQclient or server makes almost no difference to the application. You can build a client/server application with IBM MQ clients or servers.
* An application can run on the same computer as the queue manager, or on a different one. If it runs on the same computer, it is a IBM MQ server application. If it runs on a different computer, it is a IBM MQ client application. Whether it is IBM MQclient or server makes almost no difference to the application. You can build a client/server application with IBM MQ clients or servers.
* An application can run on the same computer as the queue manager, or on a different one. If it runs on the same computer, it is a IBM MQ server application. If it runs on a different computer, it is a IBM MQ client application. Whether it is IBM MQclient or server makes almost no difference to the application. You can build a client/server application with IBM MQ clients or servers.


**What tools and resources come with IBM MQ?**

* Control commands, which are run from the command line. You create, start, and stop queue managers with the control commands. You also run IBM MQ administrative and problem determination programs with the control commands.
* IBM MQ script commands (MQSC), which are run by an interpreter. Create queues and topics, configure, and administer IBM MQ with the commands. Edit the commands in a file, and pass the file to the runmqsc program to interpret them. You can also run the interpreter on one queue manager, which sends the commands to a different computer to administer a different queue manager.
* The Programmable Command Format (PCF) commands, which you call in your own applications to administer IBM MQ. The PCF commands have the same capability as the script commands, but they are easier to program.
* Sample programs.
* IBM MQ Console - web based administration (see IBM MQ Web - Lab 5)
* On Windows and Linux x86 and x86-64 platforms, where you can run the following utilities:
    * The IBM MQ Explorer. The explorer does the same administrative tasks as the script commands, but is much easier to use interactively.
    * The Postcard application to demonstrate messaging and verify your installation.
    * Tutorials.

[View an introduction to IBM MQ](https://ibm.box.com/s/gkflgmtlsq1ipihj0rkljgi564pdh9wq)

[Continue to Setup Your Environment](mq_basic_pot_envsetup.html) 

[Return MQ Basic Menu](mq_basic_pot_overview.html)