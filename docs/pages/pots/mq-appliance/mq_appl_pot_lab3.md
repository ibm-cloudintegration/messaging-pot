---
title: Exploring the IBM MQ Appliance Web UI and MQ Console
toc: false
sidebar: labs_sidebar
folder: pots/mq-appliance
permalink: /mq_appl_pot_lab3.html
summary: Web UI and MQ Console 
applies_to: [developer,administrator]
---



# Lab 3 - Exploring the IBM MQ Appliance Web UI and MQ Console

## Overview 


* What is the IBM MQ Appliance Web UI and MQ Console? 

* Starting the MQ Appliance Web UI

* Exploring the Appliance Administration Web UI

* The Appliance Administrator role

* The MQ Console dashboard

* Clean Up


## What is the IBM MQ Appliance Web UI and MQ Console?

The IBM MQ Appliance M2002 provides a browser-based administrative
interface for managing the appliance. This browser-based Web UI, which
you saw briefly in Lab 1, supplies a user-friendly alternative to the MQ
Appliance command shell for appliance administration. It also includes
the MQ Console, which provides an alternative to the MQ Appliance mqcli
command shell.

As a browser-based tool, the Web UI and MQ Console offers certain
advantages over command shell or eclipse-based tools like the MQ
Explorer, such as avoiding the overhead of installing and maintaining
software remotely, as well as being available across a much wider
variety of platforms and devices.
  
  {% include note.html content="The IBM MQ Console does not presently offer all of the configuration functionality of MQ Explorer, so there are cases where it will need to be used in conjunction with MQ Explorer or MQ Script Commands (MQSC)." %}
  
This lab will explore some of the appliance administration capabilities
of the Web UI, but the majority of this lab will focus on the
capabilities of the MQ Console. The MQ Console provides both
configuration and monitoring capabilities. In this lab, you will explore
its configuration capabilities; in a subsequent lab, you will explore
some of the monitoring capabilities it provides.

The IBM MQ Console was designed to allow users to create a more
customized experience for monitoring and administering IBM MQ. Two main
concepts to keep in mind are: 

> **Dashboards** represent the presentation space that users create. These are highly customizable and can be configured to suit a user's individual tastes. 

> **Widgets** represent the object types being displayed on the dashboard. Each dashboard tab can hold several widgets, arranged in a grid.

The design of the MQ Console supports the creation of multiple
dashboards, enabling different views of the MQ resources being hosted by
the MQ Appliance. For example, dashboards can be created that offer
views for different business applications that use MQ, with widgets
showing the objects relevant to each application. Alternatively, you
could have a tab focused purely on monitoring, with charts showing the
consumption of various resources over time (monitoring capabilities will
be explored in a subsequent lab). Widgets representing MQ objects can be
added, viewed, and deleted from the dashboard as needed. In addition,
some of the properties of the MQ objects represented by these widgets
can be modified.

In this lab, you will explore various capabilities of both the Appliance
Admin interface as well as the IBM MQ Console.

## Starting the MQ Appliance Web UI

For this lab, you should use the same CSIDE environment that you created
for Lab 1. The virtual appliance you will use for this lab will be
**MQAppl1** and also the **Windows 10 x64** VM.

{% include important.html content="It is assumed that Lab 1 has been completed. You must either complete Lab 1 before attempting this lab, or see the *MQ Appliance PoT 9.1.4 Configured - ready for HA* CSIDE template. Screen shots are from after Lab 2 completion. If Lab 2 has not been completed on the virtual appliance you are using, the results you see will differ from the examples in this lab guide." %}

### Connect to the MQ Appliance

1.  In Lab 1, you displayed the IP addresses available on virtual
    appliance MQAppl1. If you still have those addresses, you can skip
    this step.

    To obtain the IP addresses available on virtual appliance *MQAppl1*,
    issue the **show ipaddress** command (remember you must be at the high level shell). Take note of the IP addresses
    shown.

    Use the IP address for **eth0** to access the appliance using the
    Web UI.

    ![](./images/pots/mq-appliance/lab3/image164.png)

2. Double-click the Firefox icon on the Windows image to start a web
    browser session (login as **ibmdemo** / **passw0rd** if necessary).

	![](./images/pots/mq-appliance/lab3/image166.png)    

3. Navigate to **https://\<address of eth0\>:9090**. 

	If you receive any exception messages when opening the console URLs, add an exception and continue.
    
4. The login screen for the IBM Appliance web UI will be displayed.

    ![](./images/pots/mq-appliance/lab3/image169.png)

5. Enter **admin** and **passw0rd** for the user name and password,
    and click **Login**.

    ![](./images/pots/mq-appliance/lab3/image171.png)
    
    
## Exploring the Appliance Administration Web UI 

### Getting started with the Web UI 

1. Take a few minutes to explore the main features of the MQ Appliance
    web UI.

    ![](./images/pots/mq-appliance/lab3/image173.png)
 
	{% include note.html content="The MQ Appliance 'Appliance Administration' and 'MQ Administration' are separate roles. Neither is a superset or subset of the other. " %}
  
2. Notice the drop downs at the very top on the grey bar. This is the
    one part of the Web GUI that does not change when you select
    different options.

    ![](./images/pots/mq-appliance/lab3/image174.png)

3. The "admin" shown on the bar is the user name logged into the Web
    GUI. When you click the drop down arrow here, you get the option to
    logout. The circled question mark is the help menu. Clicking the
    drop down arrow here gives you options to get to the Knowledge
    Center, the Support Portal, or to generate a "DPOS" error report
    that contains configuration information, trace snippets, etc. for
    the 'system' level processes on the box.

    ![](./images/pots/mq-appliance/lab3/image175.png)

4. Next, you see that there are quite a few options along the left
    edge of the interface, in the navigation bar.

    ![](./images/pots/mq-appliance/lab3/image176.png)

5. When you log in you start in the top function, the MQ Console. It
    is highlighted to let you know that you are in the MQ Console page.
    When you hover your pointer over an icon, the name of the option
    displays.

    ![](./images/pots/mq-appliance/lab3/image177.png)

## The Appliance Administrator role

### Explore the Appliance Administration options

The **appliance administrator** manages the appliance as a whole. A
    person in that role is managing the platform, not MQ resources.
    Before going into detail on the MQ Console, which is where an MQ
    administrator would spend all or the vast majority of their time in
    the Web GUI, let us first look at each of the other options that
    appliance administrators would largely use.

### Explore the Status page

1. Click the next icon down from MQ Console, the **Status** icon.

    ![](./images/pots/mq-appliance/lab3/image178.png)
    
2. The Status page allows you to see the status of a large variety of
    components, including some appliance logs, active users (in the
    "Main" menu), date and time (in "Main"), appliance configuration,
    system components, network details, some MQ overview details, and a
    few other items. Again, this is status information -- not where you
    configure anything. Note the ability to search for a component to
    retrieve status for, and the ability to refresh the information.

    Feel free to look around the various options in this and all the
    panels.

    ![](./images/pots/mq-appliance/lab3/image179.png)

3. Click the twisty for **View Logs**. 

    ![](./images/pots/mq-appliance/lab3/image180.png)

4. You then see the various options under the View Logs category. Now
    click **System Logs**. The page will now display the list of log
    records in the system log. We do not care right now what is in the
    logs. Click the **question mark** symbol.

    ![](./images/pots/mq-appliance/lab3/image181.png)
    
5. This brings up a help popup box explaining this option. Click the
    **X** to close the window.

    ![](./images/pots/mq-appliance/lab3/image183.png)

### Explore the Network page

1. Click the **Network** icon. The Network page allows you to manage
    the interfaces on the appliance, the network, and the SSH/web
    management services. Again, you can search for a specific option,
    refresh the screen, filter the list, and perform actions on a
    selected item. Help is available for each option.

    ![](./images/pots/mq-appliance/lab3/image184.png)

2. Click the twisty for **Interface** and then the **Ethernet
    Interface** option.

    You should recognize the Ethernet interfaces you configured in
    Lab 1. Feel free to explore but do not change anything.

    ![](./images/pots/mq-appliance/lab3/image186.png)

### Explore the Administration page

1. Click the **Administration** icon. The Administration page allows
    you to manage various aspects of the appliance configuration,
    including admin users, as well as perform some network
    troubleshooting.

    ![](./images/pots/mq-appliance/lab3/image187.png)

2. Click the **Access** twisty and then the **User Account** option.
    These are the administration user accounts, not messaging users.
    Messaging users cannot currently be defined in the Web GUI.

    ![](./images/pots/mq-appliance/lab3/image189.png)

3. Now click the **Device** twisty and then the **System Settings**
    option. Briefly review the System Settings that are managed by the
    appliance administrator. Note that none of these are MQ-related.

    ![](./images/pots/mq-appliance/lab3/image191.png)

4. You can hover over
    the![](./images/pots/mq-appliance/lab3/image192.png) hotspot next to each property to see a
    description of that property. Of course, clicking the question mark
    provides help for the whole page.

    Note that some properties are greyed out, and are read-only -- these
    either are fixed, or can only be modified using the appliance's
    command-line interface.

    ![](./images/pots/mq-appliance/lab3/image193.png)

5. Make a small change to the configuration.

    In the **Comments** field, enter a comment of your choice.

    Notice that the **Apply** button is no longer greyed out.

    ![](./images/pots/mq-appliance/lab3/image194.png)

6. There is an Apply button at both the bottom of the page and the
    top. Click **Apply** at the top of the page to apply the change.

7. Click **System Settings** again. This (or clicking on any other
    option) forces the interface to recognize the need to save the
    changes there were made. Click **Review changes** to see a window
    detailing the changes that will be made.

    ![](./images/pots/mq-appliance/lab3/image195.png)

8. The window shows an entry for the change, showing what property is
    being changed (the "Comments" property in the "System" settings on
    the "System Settings" group), shows the "From" (which was blank) and
    "To" values, and the type of modification (in this case it is being
    "added" since it was blank -- as opposed to). Close the window.

    ![](./images/pots/mq-appliance/lab3/image196.png)

9. Now click **Save changes**.

    ![](./images/pots/mq-appliance/lab3/image197.png)

10. The screen will show a progress box while the change is saved, and
    then refresh to show the current, updated, settings.

    ![](./images/pots/mq-appliance/lab3/image198.png)

	When using the various administration tools, it is important to
    remember to click the "*Save changes*" button to actually make the
    change.

11. Within the Administration page is the File Management tool. This is
    very useful for copying files on (such as certificates or firmware)
    and off (logs, FDCs, backups, etc.) the appliance. This is where
    within the GUI you can access error logs (in the mqerr: URI), etc.
    
    Click the **Main** twisty and then click **File Management**.
    Folders that have content are indicated with a plus sign on the
    folder, such as ![](./images/pots/mq-appliance/lab3/image199.png)

    ![](./images/pots/mq-appliance/lab3/image200.png)

12. Notice the ability to delete, copy, rename, and move checked files
    as needed. You should be careful with this access to be sure you do
    not lose a file (move it accidentally or delete it) that you need.

    ![](./images/pots/mq-appliance/lab3/image201.png)

13. Click the **mqerr:** folder, then the **qmgrs** folder, and finally
    the **QM1** folder. Click one of the .LOG files that has content (a
    non-zero size), such as in the below **AMQERR01.LOG**.

    ![](./images/pots/mq-appliance/lab3/image202.png)

14. Since it is a text file, we can open and view the file. The file
    opens in a new tab in the browser. Close the browser tab and return
    to the MQ Appliance Web GUI.

    ![](./images/pots/mq-appliance/lab3/image203.png)

15. If you want to save any file on your system, then you will use the
    standard browser facilities to save a file. Right click the same
    file you opened in the previous step (AMQERR01.LOG), then click
    **Save Link As...**.

    ![](./images/pots/mq-appliance/lab3/image204.png)

16. Select the **Local Disk (C:)** as the destination, then click
    **Save** to save the file on your local system. This is how you can
    save backup files, logs and FDCs, etc. off the appliance.

    ![](./images/pots/mq-appliance/lab3/image205.png)

17. If you wanted to upload a file, such as a new firmware image into
    the **image:** folder, you would click **Actions...** in the Action
    column within the row representing the folder where we want to
    upload to, and then click **Upload Files**. That opens a dialog that
    allows you to browse files from your computer, to select to upload to
    the appliance. You will see this in another lab.

    ![](./images/pots/mq-appliance/lab3/image206.png)

	Other files offer different actions, such as "Edit" where
    appropriate.

### Explore the Objects page

1. Click the **Objects** icon. The Objects page allows you to manage
    all the appliance objects. Note some objects here are duplicated in
    other options, but there are some specific objects that are only
    found here too. 
    
2. Click the **Device Management** twisty and then
    click **REST Management Interface**. You notice that the SSH Service
    and Web Management Service options are duplicated, as they can be
    accessed in the Network page under Management. However, this is the
    only place you can access the REST Management Interface.

    ![](./images/pots/mq-appliance/lab3/image207.png)

    All the configuration settings reviewed in this section apply to the
    M2002 appliance as a whole.

    In later labs, you will return to some of these appliance
    administrator views and explore these in more detail.

    The rest of this lab will focus on the MQ administrative role of the
    MQ Console.
    

## The MQ Console dashboard


### Explore the MQ Console

1. Now let us return to the MQ Console. Click the **MQ Console** icon
    in the Web GUI. Let us now complete exploring the functionality in
    this page.
    
    ![](./images/pots/mq-appliance/lab3/image209.png)

2. Notice the buttons across the top right of the page:

    ![](./images/pots/mq-appliance/lab3/image210.png)

3. The checkmark on the High Availability button is the quick signal
    that high availability is configured and working, meaning that both
    appliances in the HA group are running. Click the **High
    Availability** button.
    
    {% include important.html content="What you see here depends on what labs and/or template you have completed -- whether you have created the HA group or not. " %}

    At the top of the drop down, you will get the status of the HA
    group. In this case, it is showing both appliances being online. If
    one appliance was shutdown or suspended from the HA group, you would
    see it show the status appropriately. The other options here are to
    get the details on the HA queue manager status and to show if the
    queue managers are in a normal state or if they are partitioned.
    Assuming the appliance is online, you can suspend the appliance from
    the group. If the appliance is suspended, you have the option to
    resume the appliance. You also have the option there to delete the
    HA group. You can also manage the 'keys' for the HA group.

    ![](./images/pots/mq-appliance/lab3/image212.png)

4. Click the **Disaster Recovery** button. Here are options to get the
    DR queue manager status and where you can use the GUI to create the
    DR secondary (more on this in the DR lab).

    ![](./images/pots/mq-appliance/lab3/image214.png)

5. Click the ![](./images/pots/mq-appliance/lab3/image214a.png) dashboard settings menu button. This provides the ability
    to import, export, and reset your dashboard.

    ![](./images/pots/mq-appliance/lab3/image215.png)

6. The gear cog icon allows for diagnostics. Clicking that option
    opens a dialog where you can turn on the Auto refresh for the Queue
    Managers and enable traces.

    ![](./images/pots/mq-appliance/lab3/image217.png)

7. Reset the dashboard so that the following instructions are exact.
    Click **Reset dashboard**.
    
    ![](./images/pots/mq-appliance/lab3/image219.png)

8. Click **Reset** to confirm the reset of the dashboard.
    
    ![](./images/pots/mq-appliance/lab3/image221.png)

### Organizing the dashboard

You can organize MQ Console dashboards to have a layout that is
appropriate for the information you are displaying. Multiple widgets of
the same or different types can be created, and these can be positioned
on the dashboard in any manner you see fit. The widgets can be arranged
in a grid pattern.

1. Let us start by looking at tabs. A tab is one page on your
    dashboard. You can create as many tabs as you want on your
    dashboard. Out-of-the-box, you have one tab, named "Tab 1." Again,
    notice a drop down for the tab. Click the drop down arrow.

    ![](./images/pots/mq-appliance/lab3/image223.png)

2. The top option is where you can configure the tab and give it a
    name of your choosing, and to optionally place a description on your
    tab. Notice that you can choose how many columns, from 1 to 6, with
    a 2 column layout being the default, that this specific tab uses to
    layout your widgets (though as you'll see, when you add a widget you
    can choose how many columns the widget takes up).

    ![](./images/pots/mq-appliance/lab3/image224.png)

3. Click **Configure tab...**. In the dialog, enter **Home** for the
    Tab name and **Queue Managers Tab** for the Description. Click
    **Rename**.

    ![](./images/pots/mq-appliance/lab3/image226.png)

4. The tab now shows the new name and the description under the name.

    ![](./images/pots/mq-appliance/lab3/image228.png)

5. Clicking the **+** sign brings up a similar dialog where you can
    name the tab and optionally provide a tab description. We will add
    another tab later.

    ![](./images/pots/mq-appliance/lab3/image229.png)

6. Let us now look at widgets. By default, when you first launch the
    MQ Console after initial configuration and acceptance of the license
    agreement, there will be one widget on Tab 1 -- the Queue Managers
    widget. If you have done the previous labs, you may now have several
    additional widgets on the current tab and an additional tab.
    
    ![](./images/pots/mq-appliance/lab3/image230.png)

7. Let us review the list of widget options. Click **Add widget**.

    ![](./images/pots/mq-appliance/lab3/image232.png)

8. First, notice the **Queue manager** drop down. Click the drop down
    and notice that every queue manager that has any definition on this
    appliance shows up. We can choose to manage objects for any queue
    manager, but each widget (other than the Local Queue Managers
    Widget) is showing objects for a single queue manager that you
    specify. Leave **HAQM1** selected.

    ![](./images/pots/mq-appliance/lab3/image234.png)

9. Now review the list of the different widgets available to manage MQ
    on the appliance (you need to scroll to see the entire list). Select the **Channels** widget. The window will
    close, and the Channels Widget will display for HAQM1.

    ![](./images/pots/mq-appliance/lab3/image236.png)

10. The widget is added at the first open slot in the layout.

    ![](./images/pots/mq-appliance/lab3/image238.png)
    
11. Click on the **Pen** to the right of the Widget name. This icon only displays when you move the pointer in that area.

	![](./images/pots/mq-appliance/lab3/image240.png)

	Use this dialogue box to change the name of the Widget.

12. Change the name to **HAQM1 All Channels** and click **Rename**.
    
    ![](./images/pots/mq-appliance/lab3/image241.png)

13. Now click the **Configure widget** icon on the **HAQM1 All Channels** widget.

    ![](./images/pots/mq-appliance/lab3/image242.png)

14. This brings up the configure dialog. So, you can see the effect of
    the options, click **Show** for the **System objects** option.
    Notice the options to change which queue manager this widget is for
    and the option to select a specific type of channel to view. Click
    **Save**.

    ![](./images/pots/mq-appliance/lab3/image244.png)

### Local Queue Managers widget

1. You should see a queue manager widget already on the dashboard. If
    you do not see one on the dashboard, click **Add Widget**. Then you
    click **Local Queue Managers** to add this widget on the dashboard
    tab.

    ![](./images/pots/mq-appliance/lab3/image247.png)

2. By default, all queue managers on the appliance will be displayed.
    You should see the queue manager that was created in Lab 1 (QM1),
    and Lab 2 if completed (HAQM1 and HAQM2).

    ![](./images/pots/mq-appliance/lab3/image248.png)
    
    {% include important.html content="If QM1 does not have a status of Running, that is not a problem. You will start it shortly." %}

3. Look at the controls at the top of the widget:

    ![](./images/pots/mq-appliance/lab3/image250.png)
    
    {% include important.html content="Note that controls grey out when not available, which means that you must have a queue manager selected to take action on for controls that affect the queue manager. Also, what controls are shown depend on whether the selected queue manager is in a *running* or *stopped* state." %}

4. Let us first look at the controls available when you select a *stopped* Queue Manager. Select the **QM1** Queue Manager (if QM1 is running, now click **Stop** to stop it).

	Take note of the available controls, from left to right:

	*  The **Delete** button will delete and remove a highlighted Queue Manager

	*  The **Start** button will start the highlighted Queue Manager
 
	*  The '**...**' MORE menu will present options for *High
    Availability*, *Disaster Recovery*, *Resize* the queue manager file system, and allow you to add tabs to the view.
    
    * **Deselect** allows you to un-select any queue manager

    ![](./images/pots/mq-appliance/lab3/image251.png)

5. Now let us look at the controls available when you select a *running* Queue Manager. Select the **HAQM1** Queue Manager. 
	
	Take note of the different available controls, from left to right:

	*  The **Properties** button allows you to view and alter properties of the highlighted Queue Manager

	*  The **Stop** button will stop the highlighted running Queue Manager
 
	*  The '**...**' MORE menu will present options for *High
    Availability*, *Disaster Recovery*, *Refresh security*, *Manage authority records*, *Manage create authority records*, and allow you to add tabs to the view.
    
    * **Deselect** allows you to un-select any queue manager

    ![](./images/pots/mq-appliance/lab3/image252.png)

6. The following steps are to read through. Feel free to click on
    options and close dialogs as appropriate if you wish.

7. When you Highlight a Queue Manager, the following menu options will
    become available (again, depending on if it is running or stopped):
    
    ![](./images/pots/mq-appliance/lab3/image257.png)

8. The Properties menu item will allow you to change the properties of
    a selected Queue Manager. These are the same properties that are
    managed with typical Queue Managers. Because this is an Appliance,
    certain properties are restricted or should not be changed. (i.e.
    CCSID, SSL KDB Path, etc.).

    ![](./images/pots/mq-appliance/lab3/image259.png)

9. Use the **High Availability...** option to bring up the following
    dialog:

    ![](./images/pots/mq-appliance/lab3/image260.png)

    In this dialog, you can add the *selected* Queue Manager to an HA
    group, remove the selected queue manager from the HA group, set the
    preferred location for the selected queue manager, clear the
    preferred location for the selected queue manager, resolve a
    partitioned state for the HA queue manager, or configure a floating IP
    for the queue manager.

10. Use the **Disaster Recovery...** option to bring up the following
    dialog:

    ![](./images/pots/mq-appliance/lab3/image262.png)

    In this dialog, you can get the DR status to view further DR queue
    manager information, create the DR primary, create the DR secondary,
    make the selected queue manager the DR primary, make the selected
    queue manager the DR secondary, delete the DR primary (not the queue
    manager, remove the DR nature of it), or delete the DR secondary.
    You will learn more about MQ Appliance support for disaster recovery
    in another lab.

11. Use the **Refresh security...** option to bring up the following
    dialog:

    ![](./images/pots/mq-appliance/lab3/image264.png)

    In this dialog, you can issue a "refresh security" command to
    refresh the authorization service, connection authentication
    information, or the view of the SSL repository. You will learn more
    about MQ security in another lab.

12. Use the **Manage authority records...** option to bring up the
    following dialog:

    ![](./images/pots/mq-appliance/lab3/image266.png)

    In this dialog, for the selected queue manager, you can create,
    alter, or delete authority records, setting the desired permissions.

    ![](./images/pots/mq-appliance/lab3/image268.png)

13. Use **Manage create authority records...** to bring up the
    following dialog:

    ![](./images/pots/mq-appliance/lab3/image270.png)

    In this dialog, for the selected queue manager, you can create or
    alter authorities for an entity.

    ![](./images/pots/mq-appliance/lab3/image272.png)

14. Use **Add new dashboard tab** to add a new tab on the dashboard
    with the name of the selected queue manager and with all of the MQ
    object widgets for that queue manager created. You will look at this
    shortly, as this is a great way to create new tabs for each queue
    manager you want to manage.

15. Notice the **Search...** option, that is available when you have no Queue Manager selected.

    ![](./images/pots/mq-appliance/lab3/image274.png)

    Use this control to enter a search value, to find (in a large list)
    and effectively filter the values shown. We will test this control
    shortly too.

    In addition, take note of these widget controls on the top right of
    the widget:

    * ![](./images/pots/mq-appliance/lab3/image275.png) 	Click this control to refresh the
    widget view immediately.

    * ![](./images/pots/mq-appliance/lab3/image276.png) 	Click this control to view and edit
    widget properties (not available on the *Local Queue Managers* widget).

    * ![](./images/pots/mq-appliance/lab3/image277.png) 
   	Click this control to delete the
    widget.

    You will use these controls throughout the remainder of the labs.

16. If the **QM1** queue manager shows a Status of **Stopped**, click
    the **Start** icon on the toolbar. Be sure
    the **QM1** entry in the widget is selected when you do this.

    ![](./images/pots/mq-appliance/lab3/image278.png)

    The queue manager will show a status of Starting while it is being
    initialized.

    ![](./images/pots/mq-appliance/lab3/image280.png)

17. Let us now try out a few of these things. Enter **QM1** in the
    **Search** box on the QM1 Queue Managers widget. 
    
    {% include note.html content="Remember the search option will not display if a Queue Manager is selected." %} 
    
    Notice as soon as you type QM1, the list below is instantly filtered
    to "find" only those queue managers that contain "*QM1*" within its
    name (HAQM2 was dropped from the current list). Notice at the bottom
    that there are still three queue managers defined.

    ![](./images/pots/mq-appliance/lab3/image282.png)

18. Let us display the properties of the QM1 queue manager. Note that
    you can do this by clicking the **QM1** queue manager to select it,
    and then clicking
    the Properties control on the toolbar, or
    by double-clicking the **QM1** entry in the table:

    ![](./images/pots/mq-appliance/lab3/image284.png)

19. Examine the Properties dialog for QM1:

    ![](./images/pots/mq-appliance/lab3/image286.png)

    If you are an MQ administrator, the categories and properties will
    be familiar to you. Note the different property categories that can
    be selected on the left of the dialog. Also, note that properties
    that cannot be altered are greyed out, and that properties that can
    be altered are displayed in white.

20. Try changing one of the properties. In the General properties,
    enter a description of your choice (e.g. "Queue manager for Lab 1")
    for the **Description** property.

    ![](./images/pots/mq-appliance/lab3/image288.png)

21. Notice that once you make a change to the properties, the warning
    displays that you have unsaved changes. Click **Save**. The
    properties will be updated. Click **Close** to close the properties dialog.
    
    ![](./images/pots/mq-appliance/lab3/image289.png)

22. Display the properties again (double-click **QM1** in the list of
    queue managers) and see the updated property. Select other
    categories if you like, and review the queue manager properties in
    each. Click **Close** to close the dialog when finished.

    ![](./images/pots/mq-appliance/lab3/image290.png)

23. Now click **QM1** to select that queue manager in the Queue Managers
    widget. Then click the **...** More control. In the menu, click **Add
    new dashboard tab**.

    ![](./images/pots/mq-appliance/lab3/image293.png)

24. A new tab, named *QM1* is created and opened, with eight widgets on
    it, including the Queues, Client-connection Channels, Channels,
    Listeners, Subscriptions, Topics, Authentication Information, and
    Channel Authentication Records widgets for QM1.

    ![](./images/pots/mq-appliance/lab3/image294.png)

25. You can rearrange widgets on the tab by dragging and dropping them
    where you want them. Notice that when you hover over the title bar
    of a widget, you see a crossed arrows symbol. Click and hold down on
    the title bar for the **Channels on QM1** widget, then drag it
    diagonally to the right towards the **Client-connection** **Channels
    on QM1** widget.

    ![](./images/pots/mq-appliance/lab3/image296.png)

26. Notice as you approach the **Client-connection** **Channels on
    QM1** widget, it slides down and makes room for the **Channels on
    QM1** widget. Drop it in place there.

    ![](./images/pots/mq-appliance/lab3/image297.png)

### Queues widgets

1. Still on the newly created *QM1* tab in the dashboard, look at the
    queues widget named "**Queues on QM1**." It is configured to show
    the local Queues defined to *QM1*. You should see the queues *TEST.IN*
    and *TEST.OUT* that were created in Lab 1.

    ![](./images/pots/mq-appliance/lab3/image298.png)

2. Click the
    ![](./images/pots/mq-appliance/lab3/image276.png) Configure widget control to bring up
    the configuration of the widget dialog. 

    ![](./images/pots/mq-appliance/lab3/image300.png)

    For certain options, you can hover over
    the ![](./images/pots/mq-appliance/lab3/image302.png) symbol to access help for that option.

    Notice you can set a customized title for the widget if desired. You
    can change which queue manager the widget is configured against. You
    can optionally select a specific queue type to display, either all
    queues, or specifically only local, remote, alias, or model queues.

    ![](./images/pots/mq-appliance/lab3/image303.png)

    You again have the ability to show system objects or not. Click
    **Cancel**.

3. The toolbar for the MQ Queues widget is very similar to the toolbar
    on the Local Queue Managers widget. Note that this is the view/options when a queue is selected.

    ![](./images/pots/mq-appliance/lab3/image304.png)

    There are a few different controls: 
    
    ![](./images/pots/mq-appliance/lab3/image306.jpg) 	Delete queue 
    
    ![](./images/pots/mq-appliance/lab3/image307.jpg) 	Queue properties
    
    ![](./images/pots/mq-appliance/lab3/image308.jpg) 	Put message 
    
    ![](./images/pots/mq-appliance/lab3/image309.jpg) 	Browse messages                        

	 ![](./images/pots/mq-appliance/lab3/image310.jpg) 	More 
	 
	 Under the **...** More menu, the options are:
	 
	 *  **Clear queue...** - to clear all messages from the queue          
	 *  **Manage authority records...** - to manage authority records on the queue

	You will explore the Queues widget in more detail shortly.

### Channels widget

1. Look at the Channels widget named "**Channels on QM1**," showing
    the Channels defined to QM1. You should see the USER.SVRCONN channel
    that was created in Lab 1.

    ![](./images/pots/mq-appliance/lab3/image314.png)

2. Click the
    ![](./images/pots/mq-appliance/lab3/image276.png) Configure widget control to bring up
    the configuration of the widget dialog.

    ![](./images/pots/mq-appliance/lab3/image317.png)

    Again, you can set a custom widget title, change the queue manager
    selected, select the specific type of channel displayed, or chose to
    show system objects. 

3. Click **Cancel**.

4. The toolbar for the MQ Channels widget is very similar to the MQ
    Queues toolbar. The controls (add, delete, properties, start, stop)
    should be self-explanatory. Note that this is the view/options when a channel is selected.

    ![](./images/pots/mq-appliance/lab3/image318.png)

5. Display the properties for the **USER.SVRCONN channel**.

	Note that you can do this by clicking the
    **Properties** icon after selecting the channel, or
    by double-clicking the *USER.SVRCONN* entry in the table.

    ![](./images/pots/mq-appliance/lab3/image321.png)

6. Review the channel properties.

    ![](./images/pots/mq-appliance/lab3/image324.png)

    If you are an MQ administrator, the categories and properties will
    be familiar to you. Recall that you can select various categories of
    properties from the left, that properties that cannot be altered are
    greyed out, and certain properties have
    a![](./images/pots/mq-appliance/lab3/image325.png) symbol to provide help for the property.

7. Feel free to explore the properties, and then click **Close**.

### Channel Authentication Records widget

1. Look at the Channel authentication records widget named "**Channel
    Authentication Records on QM1**," showing the Channel Authentication
    Records defined to QM1. You should see the CHLAUTH records for
    channel *USER.SVRCONN* that were created in Lab 1.

    ![](./images/pots/mq-appliance/lab3/image326.png)

2. Click the
    ![](./images/pots/mq-appliance/lab3/image276.png) Configure widget control to bring up
    the configuration of the widget dialog.

    ![](./images/pots/mq-appliance/lab3/image328.png)

    You have the standard widget options discussed previously.

3. Click **Cancel**.

4. The toolbar for the MQ Channel Authentication Records is very
    similar to the other MQ Object toolbars. The controls (Create, Delete,
    Properties) should be self-explanatory.

    ![](./images/pots/mq-appliance/lab3/image330.png)

5. Display the properties for the CHLAUTH record for *USER.SVRCONN*.

6. Note that you can do this by clicking
    the **Properties** icon after selecting the record, or
    by double-clicking the CHLAUTH record in the table.

    ![](./images/pots/mq-appliance/lab3/image333.png)

7. Review the channel properties.

    ![](./images/pots/mq-appliance/lab3/image334.png)

8. Feel free to explore the properties, and then click **Close**.

### Creating a dashboard

The design of the MQ Console supports the creation of multiple
dashboards, enabling different users to create dashboard layouts that
offer views for different business applications that use MQ, or that
serve different roles (for example, a dashboard that is focused purely
on monitoring). Alternatively, you can create dashboards that simply
reflect your personal preference.

When dashboards are created by an administrator, they are "remembered",
allowing the user to access them later, from the same or different
browser, without needing to recreate them. Dashboard layouts can also be
exported and imported, allowing these views to be shared by different
users.

In this section, you will create a new dashboard, create new MQ queue
managers and MQ objects, and explore additional features of the MQ
Console widgets. You will also customize the dashboard to view only a
subset of the MQ resources defined on the appliance. Finally, you will
see how to export and import dashboard layouts.

1. Create a new tab on the MQ Console, by clicking the **+** sign.

    ![](./images/pots/mq-appliance/lab3/image335.png)

2. A new tab dialog will appear. Enter a name for the new tab
    ("**MyTab**" in this example) and a description ("**Lab 3 Tab"** in
    this example). Click **Add**.

    ![](./images/pots/mq-appliance/lab3/image336.png)

3. A new empty tab is created with the given name and description
    shown.

    ![](./images/pots/mq-appliance/lab3/image338.png)

4. Add a new *Local Queue Managers widget*. Click **Add widget**.

5. On the add widget dialog, click **Local Queue Managers**.

    ![](./images/pots/mq-appliance/lab3/image342.png)

6. The Local Queue Managers widget will appear. As before, it displays all
    the queue managers on the appliance.

    Click the **Create** 
    icon on the widget toolbar.

7. Enter **QM1** for the name. Notice that the MQ Console notifies you
    that this is a duplicate name, and will not let you click the *Create*
    button.

    ![](./images/pots/mq-appliance/lab3/image345.png)

8. Enter **MY_QM** for the queue manager name, enter **2414** for
    port, and then click **Create**.

    ![](./images/pots/mq-appliance/lab3/image347.png)

9. You should see the following error when trying to create the queue
    manager:

    ![](./images/pots/mq-appliance/lab3/image349.png)

10. When an error occurs while creating or working with MQ objects
    using the MQ Console, the failing commands and reason codes will be
    displayed.

    In the example shown, there was not enough file system storage on
    the virtual appliance to create another queue manager.

    By default, the MQ Appliance allocates a 64GB file system for each
    queue manager that is created. The file system size allocated to a
    queue manager is configurable.

11. Click the **X** at the **END** of the error message to close the
    message box.

    ![](./images/pots/mq-appliance/lab3/image351.png)

12. Click
    the **Create** icon on the widget toolbar again.
    This time, enter **MY_QM** for the queue manager name, enter
    **2414** for port, and enter **2** for the file system size. Notice the ability to select
    automatic startup mode, meaning the queue manager will start
    automatically when the appliance starts up. Select **Automatic** start up. Finally, click
    **Next**.

    ![](./images/pots/mq-appliance/lab3/image354.png)
    
13. Notice the *High availability* setting is on the second page of the *Create a Queue Manager* dialog. Leave the default of **None** and click **Create**.  
    
    ![](./images/pots/mq-appliance/lab3/image355.png)

14. It will take a few seconds for the queue manager to be created.

    You should see the new queue manager be created.
    
    ![](./images/pots/mq-appliance/lab3/image357.png)

    **If the queue manager is not successfully created, see the
    instructor.**

15. Set the filters to hide queue managers you are not interested in
    seeing.

    In the **Search** field, type **MY**. The other queue managers will
    no longer be visible in the widget.

    ![](./images/pots/mq-appliance/lab3/image358.png)
    
16. Return to the command line console for the virtual appliance.
    Ensure you are in the **mqcli** shell. Enter **dspmq**.

    You should now see that **MY_QM** exists on the appliance.

    ![](./images/pots/mq-appliance/lab3/image360.png)

17. Enter the following command to create an additional queue manager
    on the appliance:

    **`crtmqm -p 3414 -fs 1 MY_QM2`** 

    ![](./images/pots/mq-appliance/lab3/image361.png)

    You should see the new queue manager be created.

	If the queue manager is not successfully created, see the instructor.

18. Return to the Web GUI MQ Console.

    In the queue manager widget, note that the new *MY_QM2* queue manager
    is visible.

    Note: You may need to reenter the search value *MY*.

    ![](./images/pots/mq-appliance/lab3/image362.png)

    Notice that the queue manager is stopped. The default when creating
    a non-HA queue manager is for it to be in manual startup mode.

	{% include note.html content="Queue managers created using the MQ Console (such as **MY_QM** above), are both created and started *immediately*. This is different from when you create a queue manager from the appliance command line console (such as **MY_QM2** above), where you must first create the queue manager (using *crtmqm*) and then start it (using *strmqm*). " %}
  
19. You are not going to use this queue manager, so delete it.

    Click **MY_QM2** in the list, and then click the
    **Delete** toolbar control.

    ![](./images/pots/mq-appliance/lab3/image365.png)

20. Click **Delete** to confirm the deletion.

    ![](./images/pots/mq-appliance/lab3/image367.png)

21. Return to the command line console for the virtual appliance. Enter
    **dspmq**.

    You should now see that **MQ_QM2** has been deleted, but the other
    queue managers remain.

    ![](./images/pots/mq-appliance/lab3/image368.png)
    
22. Back in the MQ Console, Click **Add widget** again. On the add
    widget dialog, select the **MY_QM** queue manager from the *Queue
    manager* drop down box, and then select **Queues**.

    ![](./images/pots/mq-appliance/lab3/image369.png)

23. The Queues widget is added to the dashboard tab. Hover next to the
    Widget name and the pen will appear. Click on this.

    ![](./images/pots/mq-appliance/lab3/image371.png)

24. Enter **Queues for Application #1** in the Widget title, then
    click **Rename**.

    ![](./images/pots/mq-appliance/lab3/image373.png)

25. The new widget should have the title you specified.

    Click the **Create** icon on the widget toolbar to create a new
    queue.

    ![](./images/pots/mq-appliance/lab3/image376.png)

26. Enter a Queue Name of **App1.Q1**. Click **Create**.

    ![](./images/pots/mq-appliance/lab3/image378.png)
    
27. Now complete the following additional steps:

    a.  Create another **Queues** widget, for the **MY_QM** queue
        manager. Rename the Widget **Queues for Application #2**.

    b.  In this new widget, create a new queue. Name the queue
        **App2.Q1**.

    c.  In the **Queues for Application #1** widget, enter a **Search**
        value of **App1**.

    d.  In the **Queues for Application #2** widget, enter a **Search**
        value of **App2**.

    e.  Arrange the dashboard widgets so that the Queue Manager widget
        is on the left and the Queue widgets are on the right.

28. When the above is complete, your dashboard should look something
    like this:

    ![](./images/pots/mq-appliance/lab3/image380.png)

    This provides a simple example of how a custom dashboard can be
    created.

### Exporting and importing dashboards

Dashboard layouts can be exported and shared with others. This section
will demonstrate how this is done.

1. In the upper right corner are controls that allow you to import,
    export, and reset the layout of the current dashboard.

    ![](./images/pots/mq-appliance/lab3/image383.png)

2. To export the current dashboard layout, click the
    ![](./images/pots/mq-appliance/lab3/image385.png) dashboard settings menu, then click
    **Export dashboard**.

    ![](./images/pots/mq-appliance/lab3/image386.png)

3. Note the filename ("**MQConsole.json**"). Make sure **Save File**
    is selected, and click **OK**.

    ![](./images/pots/mq-appliance/lab3/image388.png)

4. Delete the current dashboard tab. Click the drop down arrow to the
    right of **MyTab**, then click **Delete tab...**.

    ![](./images/pots/mq-appliance/lab3/image390.png)

5. Click **Delete** to confirm.

    ![](./images/pots/mq-appliance/lab3/image392.png)

6. Now select **Import dashboard**.

    ![](./images/pots/mq-appliance/lab3/image394.png)

7. Click **Browse** to locate the configuration file that was
    previously saved.

    ![](./images/pots/mq-appliance/lab3/image396.png)

8. Select **Downloads** on the left, then locate and select the
    previously saved file **MQConsole.json**, and click **Open**.

    ![](./images/pots/mq-appliance/lab3/image398.png)

9. Select the **Replace existing dashboard with imported dashboard
    tabs** option, and then click **Import**.

    ![](./images/pots/mq-appliance/lab3/image400.png)

10. You should see your saved dashboard **MyTab** restored, with the
    widgets and the layout that were in place when the dashboard was
    saved. Click **MyTab** to view it.

    ![](./images/pots/mq-appliance/lab3/image403.png)

11. One thing to note is that search filters are not considered part of
    this dashboard layout, and so are not saved when the dashboard is
    exported. Re-enter the *Search* values as shown below:

    ![](./images/pots/mq-appliance/lab3/image406.png)

### Working with queues

This section will explore some of the capabilities available when
working with queues using the MQ Console.

1. On the **Queues for Application #1** widget, click the **App1.Q1**
    queue, then click
    the![](./images/pots/mq-appliance/lab3/image408.png) Properties icon in the toolbar.

    ![](./images/pots/mq-appliance/lab3/image409.png)

2. Review the queue properties. Again, note the various categories to
    select on the left. Click **Close** when complete.

    ![](./images/pots/mq-appliance/lab3/image411.png)

3. Put a message on the **App1.Q1** queue.

    To do this, on the **Queues for Application #1** widget, click the
    **App1.Q1** queue.

    Then click the
    ![](./images/pots/mq-appliance/lab3/image308.jpg) (Put message) icon on the toolbar.

    ![](./images/pots/mq-appliance/lab3/image414.png)

4. Enter some test data and click **Put**.

    ![](./images/pots/mq-appliance/lab3/image416.png)

5. Put two more messages on the same queue, with different message
    content. When you are done there should be three messages total, as
    shown below.

    ![](./images/pots/mq-appliance/lab3/image418.png)

6. Click the **App1.Q1** queue in the widget, and then the
    ![](./images/pots/mq-appliance/lab3/image309.png) (Browse messages) icon in the toolbar.

    ![](./images/pots/mq-appliance/lab3/image421.png)

7. A *Browse Messages* dialog will be displayed.

    From this dialog, you can view messages on the queue and sort them
    by message content and put date and time. If there are many
    messages, you can use Search filters to reduce the number of
    messages displayed. Click **Close** when done trying sort options.

    ![](./images/pots/mq-appliance/lab3/image423.png)
    
## Clean Up


1. To clean up, still on the **MyTab** tab, return to the **Local Queue Managers** widget.

    Click the **MY_QM** queue manager and click the
    ![](./images/pots/mq-appliance/lab3/image426.jpg) icon.

    ![](./images/pots/mq-appliance/lab3/image427.png)

2. Click **Stop** to stop the queue manager.

    ![](./images/pots/mq-appliance/lab3/image429.png)

3. Click the ![](./images/pots/mq-appliance/lab3/image432.jpg) icon in the Queue Managers toolbar
    to delete the queue manager.

    ![](./images/pots/mq-appliance/lab3/image433.png)

4. Click **Delete** on the confirmation window.

    ![](./images/pots/mq-appliance/lab3/image435.png)

5. The **MY_QM** queue manager should be gone. Clear the **Search**
    filter to be sure of what queue managers are left.

    ![](./images/pots/mq-appliance/lab3/image436.png)

6. The widget should show the three queue managers remaining.

	![](./images/pots/mq-appliance/lab3/image437.png)

7. Click the drop down arrow on **MyTab**, then click **Delete tab...**.

    ![](./images/pots/mq-appliance/lab3/image439.png)

8. Click **Delete** on the confirmation window.

    ![](./images/pots/mq-appliance/lab3/image441.png)
    
9. Click the
    ![](./images/pots/mq-appliance/lab3/image442.png) dashboard settings menu, and then
    click **Reset Dashboard** to reset the dashboard to its default
    view.

    ![](./images/pots/mq-appliance/lab3/image444.png)

10. Click **Reset** on the confirmation window.

    ![](./images/pots/mq-appliance/lab3/image446.png)
    
11. The MQ Console should now display a single tab, named the default
    of *Tab 1*, with only a *Local Queue Managers* widget.

    ![](./images/pots/mq-appliance/lab3/image449.png)

This concludes the IBM MQ Appliance Web UI and MQ Console lab.

 