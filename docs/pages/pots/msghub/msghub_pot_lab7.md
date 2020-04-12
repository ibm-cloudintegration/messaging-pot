---
title: Monitoring Event Streams on IBM Cloud Private
toc: false
sidebar: labs_sidebar
folder: pots/msghub
permalink: /msghub_pot_lab7.html
summary: Monitoring Event Streams
applies_to: [developer,administrator]
---

# ICP Platform Monitoring and Logging Features
In this lab exercise you will learn about the monitoring options for Event Streams and to review logs for troubleshooting.

## Introduction

This lab is concerned with exploring additional monitoring and logging features of the underlying ICP platform. The primary interface which is expected to be used for monitoring is the Event Streams Toolbox UI (user interface) as this contains features which are specific and meaningful in a Kafka, and hence Event Streams, context. ICP includes the "ELK" stack (Elasticsearch, Logstash and
Kibana) components to provide a common monitoring and logging framework.
Describing the full capabilities of the ELK stack components are beyond
the scope of this lab, but there are various articles and tutorials on
the internet if you have an interest or need to explore more deeply.

These ELK related features of ICP are useful to understand to a basic level and which deserve some attention for the following situations:

*  It is intended to exploit the ELK stack for monitoring as a
    preferred and consistent approach. This may be the case where ICP
    has been selected as the container orchestration and deployment
    platform or there may already be experience is configuring and using
    ELK stacks.

*   Where trying to do problem determination by examining and searching
    information in the logs

## Monitoring

The monitoring framework is built around Grafana which allows features
for rich customization of monitoring dashboards which may either be
included with an ICP distribution, as samples with products available in
the ICP catalog or created by the customer based on their specific
requirements. 

{% include note.html content="At the time of writing, an example sample dashboard was not included in the IBM Event Streams GA helm chart distribution, but can be obtained from a github repository. The following instructions describes the latter approach." %}

1.  From the navigator, select ***Platform -\> Monitoring:***

	![](./images/pots/msghub/lab7/image17.png)

2.  The Grafana interface opens in another browser tab/window. An
    example / sample dashboard for Event Streams is made available which
    you can import into Grafana.

3.  You can obtain the sample dashboard definition from github at:

	<https://github.com/IBM/charts/blob/master/stable/ibm-eventstreams-dev/ibm_cloud_pak/pak_extensions/dashboards>

4.  Either download the file ibm-eventstreams-grafanadashboard.json to
    your environment or copy it to the clipboard.

5.  From the home page, click on the "+" sign and the **Import**:

    ![](./images/pots/msghub/lab7/image22.png)

6.  Either upload the JSON file or paste the contents into the importer
    and press Load:

    ![](./images/pots/msghub/lab7/image23.png)

7.  Select and open the IBM Event Streams dashboard.

8.  Ensure that the Event Streams dashboard specifies the correct
    *namespace* and *release* for your deployed environment: 
    
    {% include warning.html content="If you have done the previous labs, your release should be eslab, not esv1 as shown in the screen shot." %}

    ![](./images/pots/msghub/lab7/image18.png)

9.  The dashboard will initially start with default/null values until it
    refreshes with the latest data, and will then appear as follows:

    ![](./images/pots/msghub/lab7/image19.png)

10. Use one of the dashboard window panes to explore the layout. In the
    top right, you will see the display interval. In the left you will
    see an Information indicator which, when hovered over will display
    any contextual help information you supply. Clicking on the title
    allows you to manipulate the pane, including the ability to edit its
    definition. 
    
    ![](./images/pots/msghub/lab7/image21.png)

11. Let's try to create a new panel to monitor another metric available
    in Kafka. From the dashboard, select the ***Add Panel*** button in
    the top-right: 
    
    ![](./images/pots/msghub/lab7/image24.png)

12. Select the ***Graph*** button: 

	![](./images/pots/msghub/lab7/image25.png)

13. An empty panel is displayed. Click on the title bar to configure it:

    ![](./images/pots/msghub/lab7/image27.png)

14. In the first query input field, start typing "kafka" and a drop-down
    list of the metrics with names starting with "kafka" are displayed.
    These are the metrics which are provided with the Kafka
    distribution. As an example, select the one shown below to display
    in the panel:

    ![](./images/pots/msghub/lab7/image28.png)

15. Switch to the General tab of the Graph definition. Set a title for
    the panel. Then return to the dashboard by clicking on the arrow in
    the top right:

    ![](./images/pots/msghub/lab7/image30.png)

16. Your basic panel is now displayed:

    ![](./images/pots/msghub/lab7/image31.png)

17. If you wish, take a few minutes to explore, or experiment with, the
    other aspects of the Grafana interface.

## Logging


The logging framework of ELK is typically accessed either:

*   by navigating from a selected component in the ICP console, where
    the query will have been constructed automatically so that only the
    logging information for that component will be displayed.

*   by launching the Kibana discovery dashboard where you can build or
    run previously saved queries to examine logging information,

In the first case, you may have observed that the Event Streams user
interface is reporting a specific pod (or container) has some issues, so
you want to examine the logging information to understand what may be
occurring:

1.  From the navigator, select ***Workloads -\> Helm Releases***.

18. Find the deployed release of Event Streams:

    ![](./images/pots/msghub/lab7/image8.png)

19. Click on the name to display the list of details for the release.
    Scroll down to the section for ***Pods***.

    ![](./images/pots/msghub/lab7/image9.png)

20. Locate the pod you wish to examine logging information for and
    select View Logs. 
    
1.  Kibana opens and if this is the first time in Kibana, you will need to setup the index pattern. From the drop down select “@timestamp:” and then press create. 

	![](./images/pots/msghub/lab7/image10a.png)

1.  Close the browser window and re-launch from the “View Logs” link.
    
1.  The Kibana discovery dashboard opens in a new
    browser tab/window:

    ![](./images/pots/msghub/lab7/image10.png)

21. Note that the name of the pod was already set in the search window
    and that this has caused a query to execute and return matching
    logging information.

22. Click on the time selector at the top right of the window. You will
    see that there is fine-grained control of how to select the period
    of monitoring data you wish to display:

    ![](./images/pots/msghub/lab7/image11.png)

23. If you click on the twisty to the left of a specific log entry, it
    will expand to show the individual fields:

    ![](./images/pots/msghub/lab7/image12.png)

24. These fields can be used in search filter expressions to reduce the
    returned data or to search for specific entries. The logging
    information is derived from output from the *stdout* or *stderr*
    stream. As an example, let's say you wanted to focus on errors
    written to the *stderr* stream. So, to filter on that, click on
    ***Add a filter*** and scroll down and select ***stream***:

    ![](./images/pots/msghub/lab7/image13.png)

25. Then build the query expression and click ***Save***:

    ![](./images/pots/msghub/lab7/image14.png)

26. Observe that the result set is reduced accordingly.

	In the second case, you wish to start the Kibana user interface

1.  From the navigator, select ***Platform -\> Logging***

    ![](./images/pots/msghub/lab7/image15.png)

2.  The Kibana interface opens but note that no search information is
    supplied, so, by default, you will be viewing all logging
    information. Try to build a filter expression (using the previous
    examples) to restrict the returned log information based on one or
    more fields and then to look in the detail log fields for something
    of interest. Suggest using warnings as shown below.

    ![](./images/pots/msghub/lab7/image16.png)

3.  If you wish, take a few minutes to explore, or experiment with, the
    other aspects of the Kibana interface.

## Metering

Although this is not strictly a monitoring or logging feature, the
metering service uses the underlying captured data to report on the
usage of the components which may be used for external license reporting
or for internal purposes. To explore and report of the usage of your
Event Stream deployment:

1.  From the navigator, select ***Platform -\> Metering***:

    ![](./images/pots/msghub/lab7/image1.png)

27. The metering *Groups* are displayed. Select ***Namespaces*** and
    then select the namespace used in your Event Streams deployment:

28. Notice that the chargeable components (ie containers) are listed
    separately and that there are many containers which IBM does not
    license and charge for. Select the IBM Event Streams (Chargeable)
    containers:

    ![](./images/pots/msghub/lab7/image2.png)

29. You will see each Event Streams Kafka container shown. In a default
    installation, you will have 3 containers deployed. Select one of
    them. The Usage tab shows the CPU deployment over the selected
    period:

    ![](./images/pots/msghub/lab7/image3.png)

30. The Details tab shows data identifying the deployed configuration
    (Software tab) and the OS (Environment tab):

    ![](./images/pots/msghub/lab7/image4.png)

    ![](./images/pots/msghub/lab7/image5.png)

31. Click on ***View all*** to go back to the complete list.

    ![](./images/pots/msghub/lab7/image6.png)

32. You can export a file containing the deployment by CPU. Click on
    ***Export to CSV*** and observe the output:

    ![](./images/pots/msghub/lab7/image7.png)

33. If you want to obtain a full deployment report, then click on
    Download Report and review the output:

    ![](./images/pots/msghub/lab7/image32.png)

## Congratulations
You have learned how to monitor Event Streams and view the logs.  

[Continue to Lab 8 - Event Streams Geo-Replication](msghub_pot_lab8.html)