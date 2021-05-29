---
title: IBM MQ Overview
toc: false
sidebar: labs_sidebar
folder: pots/mq-cp4i
permalink: /mq_cp4i_pot_overview.html
summary: Introduction to IBM MQ
applies_to: [developer administrator]
---

## Overview

This guide and associated standard Virtual Server Instance (VSI) help you get started with IBM MQ on IBM Cloud Pak for Integration (CP4I). This is not a primer for MQ basics. The intent is to show how to create and operate MQ queue managers on IBM Cloud Pak for Integration running on a Red Hat OpenShift cluster.

You will learn how to create a simple queue manager, a multi-instance queue manager for high availability, an MQ cluster using the latest MQ feature Uniform Clusters, and using a Tekton pipeline to make updates to a queue manager for continuous integration.

For the MQ on CP4I PoT, multiple users will create each of the above sharing one CP4I on a single Red Hat OpenShift Kubernetes Service (ROKS) cluster. 

### Names used in lab

| student number | QM prefix | Lab 2 QM Name | Lab3 QM Name | Lab4 QM Names | Lab5 QM Name | 
|:--------------:|:---------:|:------------:|:------------:|:-------------:|:------------:|
| 01             | mq01      | mq01qs       | mq01mi       | mq01a, b, c   | mq01qmp      |
| 02             | mq02      | mq02qs       | mq02mi       | mq02a, b, c   | mq02qmp      |
| 03             | mq03      | mq03qs       | mq03mi       | mq03a, b, c   | mq03qmp      |
| 04             | mq04      | mq04qs       | mq04mi       | mq04a, b, c   | mq04qmp      |
| 05             | mq05      | mq05qs       | mq05mi       | mq05a, b, c   | mq05qmp      |
| xx...          | mqxx...   | mqxxqs...    | mqxxmi...    | mqxxa, b, c...| mqxxqmp...   |


[View an introduction to IBM MQ](https://ibm.box.com/s/gkflgmtlsq1ipihj0rkljgi564pdh9wq)

[Continue to Setup Your Environment](mq_cp4i_pot_envsetup.html) 

[Return MQ CP4I Menu](mq_cp4i_pot_overview.html)