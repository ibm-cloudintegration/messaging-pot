---
title: IBM MQ Advanced Message Security Overview
toc: false
sidebar: labs_sidebar
folder: pots/mq-ams
permalink: /mq_ams_pot_overview.html
summary: Introduction to MQ AMS
---

## Overview

This guide and associated VMware image(s) helps you get started with IBM MQ Advanced Message Security (AMS) on distributed platforms.

IBM MQ Advanced Message Security allows WebSphere MQ applications   to send sensitive data, such as high-value financial transactions and personal information, with different level of protection by using a public key cryptography model. The product provides a high level of protection for sensitive data flowing through a WebSphere MQ network, while not impacting the end applications.

IBM MQ Advanced Message Security performs the following functions:  

* Secures sensitive or high-valued transactions processed by WebSphere MQ

* Detects and removes rogue or unauthorized messages before they are processed by a receiving application  

* Insures that messages were not modified while in transit from queue to queue

* Protects data at rest (messages stored on queues)

* Secures WebSphere MQ existing proprietary and customer-written applications without requiring any changes to the applications environment

## Introduction

WebSphere MQ Advanced Message Security augments WebSphere MQ security by providing data protection of messages. Two “Qualities of Protection” (QOP) are provided:  

* Integrity (signed)  
IBM MQ messages are signed when put on a queue and the signature embedded into the message is verified when the message is retrieved. The signature identifies the sender of the message and ensures that the content of the message has not been changed.  
WebSphere MQ Advanced Message Security uses PKCS #7 data-envelope to envelop a message in a digital signature.  

* Privacy (sealed)  
IBM MQ messages are signed as above but the content of the message is also encrypted. The key used to encrypt messages is itself encrypted for each potential recipient so that the messages can be retrieved by one or more recipients.

 
Data protection is provided by various interceptors relying on configuration files, PKI keystores and certificates.  This is in addition to the standard protection available for WebSphere MQ objects using the WebSphere MQ Object Authority Manager (OAM) or SAF (via a product such as IBM’s RACF) on z/OS. 


The following APIs are supported:

* Message Queue Interface (MQI)

* Java Message Service (JMS)

* WebSphere MQ Java Service (JMS) 1.0.2 and 1.1

* WebSphere MQ Java Classes
 
 
WebSphere MQ Advanced Message Security implements message security by intercepting WebSphere MQ API calls.

Three interceptors are provided:

* IBM MQ server interceptor  
To protect messages put or get by applications using the mqm library and  
using the interprocess communication (IPC), also called bindings mode.

   This interceptor will also protect messages for Java applications (either using the MQ Java API or JMS) connecting in MQ bindings mode.

* IBM MQ C client interceptor  
To protect messages put or get by applications using the mqm client library and therefore connecting using a (non-Java) MQ client connection.

* IBM MQ Java interceptor (JMS and MQ Java)  
To protect messages put or get by Java applications (MQ Java API or JMS) using either client connections or bindings mode.
Note that if you use this interceptor for applications using Java bindings mode you should consider disabling the server interceptor to avoid signing/encrypting messages twice.

IBM MQ Advanced Message Security uses Public Key Infrastructure (PKI) identities to represent users or applications. The identities are used to sign and encrypt messages. 

An identity is represented by the distinguished name (DN) of a personal X.509 (v2/v3) certificate associated with the signed and/or encrypted message. To authenticate this authority, the user or application must have access to a keystore containing a public certificate and an associated private key.

A mechanism is provided to map the OS identity of the user or application to the specific keystore file. 

WebSphere MQ Advanced Message Security provides message security.  It does not provide authorization services for WebSphere MQ objects. The task to authorize access to WebSphere MQ resources (queue manager, queues and other objects) is the responsibility of the WebSphere MQ Object Authority Manager (OAM) or RACF (or other SAF) on z/OS. When an interceptor intercepts the MQ API calls from an application it will delegate the authorization request to the IBM MQ authority service. 

IBM MQ Advanced Message Security defines an error handling queue (SYSTEM.PROTECTION.ERROR.QUEUE) to manage messages that contain errors or messages that cannot be decrypted. If a received message does not meet the security requirements for the queue it is on, then the message is moved to the error handling queue. 

List of reasons why a message might be sent to the error handling queue:  

* Quality of protection mismatch 
* Decryption error 
* PDMQ header error 
* Size mismatch 
* Encryption algorithm strength mismatch 
* Unknown error   

The system error handling queue can optionally be defined as an alias queue pointing to another queue.

For the latest and complete list of supported platforms refer to the link below:  

[IBM MQ AMS Supported Platforms](https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_9.1.0/com.ibm.mq.ins.doc/q009140_.htm)

