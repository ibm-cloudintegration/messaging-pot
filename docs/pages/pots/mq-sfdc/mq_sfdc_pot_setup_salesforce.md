---
title: Creating a Salesforce Developer Account
toc: false
sidebar: labs_sidebar
folder: pots/mq-sfdc
permalink: /mq_sfdc_pot_setup_salesforce.html
summary: Creating a Salesforce Developer Account
---
![](./images/pots/mq-sfdc/platform.png)

In order to complete the labs you will need a valid username and password for Salesforce.com.  Since integration is provided through the Salesforce.com APIs you will also need to know the Salesforce.com security token associated with the username.  Ensure that you have created your Salesforce Developer account before starting the labs.The following section will take you through Setting up an account, activating the access to your Salesforce account and obtaining the security token. You should do this from within the Proof of Technology virtual machine and not from the native PC.If you already have a developer account and know this information then you will still need to complete the steps found in the [Accessing Salesforce From Your Workstation](#access) section to enable access your Salesforce account. ## Setting up a Salesforce Developer Account

1. Use a web browser to navigate to the following URL and then click on the **Sign up** button: [https://developer.force.com/](https://developer.force.com/)  
![](./images/pots/mq-sfdc/salesforce_login_page.png)

2. You will need a valid email address that you can access from the lab image in order to complete the application.  Complete all of the fields and ensure you select the checkbox to agree to the terms and conditions.  Click on the **Sign me up** button to proceed. 
 
	![](./images/pots/mq-sfdc/salesforce_new_account.png)

3. You should receive an email from Salesforce.com. Sign on to your email account in order to review the registration email.  The email will contain a subject line similar to the following: **Salesforce.com login confirmation**. Click on the **Verify Account** button found in the email and then perform the required steps to complete the account setup. 

	![](./images/pots/mq-sfdc/salesforce_change_password.png)
	
4. The Salesforce Home screen for developers will appear.  Log out of this site by clicking on the avatar in the upper right corner of the page and then clicking on the **Log Out** link.

	![](./images/pots/mq-sfdc/salesforce_logout_developer.png)
  
5. You now have access to the developer account.  Verify access by continuing with the following steps to view a set of sample accounts.

6. Navigate to the [Salesforce.com](https://login.salesforce.com) website.  Login using your developer account.

	![](./images/pots/mq-sfdc/salesforce_login.png)

7. Accessing Salesforce.com from a workstation that you have not used before may cause Salesforce to prompt you to verify your identity as you log on.  If prompted, enter the verification code that was sent to your email address.

	![](./images/pots/mq-sfdc/salesforce_verify_identity.png)

8. You may also be prompted to register your mobile phone.  This step is optional.  You may enter your mobile phone and click on the **Register** button or simply click on the **Remind Me Later** or **I don't want to Register My Phone** links to proceed.  

	![](./images/pots/mq-sfdc/salesforce_register_mobile_phone.png)
	
9. The initial Salesforce Setup page will be displayed.  Click on the avatar in the upper right corner of the page and then click on the **Switch to Salesforce Classic** link to change the format of the pages.

	![](./images/pots/mq-sfdc/salesforce_setup_page.png)

10. Next click on the **Accounts** tab.

	![](./images/pots/mq-sfdc/salesforce_accounts_tab.png)

11. In the **View** dropdown select **All Accounts** from the drop down list and then click on the **Go!** button. A list of sample accounts will be displayed. 

	![](./images/pots/mq-sfdc/salesforce_all_accounts_dropdown.png)
	
12. You can look at the detail for any of the accounts by clicking on an account name.

	![](./images/pots/mq-sfdc/salesforce_accounts_list.png)
	
13. Continue by completing the steps found in the [Set and Retrieve the API Security Token](#token) section. 

<a name="access"></a>
## Accessing Salesforce From Your Workstation

Accessing Salesforce.com from a workstation that you have not used before may cause Salesforce to prompt you to verify your identity as you log on:  

![](./images/pots/mq-sfdc/salesforce_verify_identity.png) 

Upon receipt of your email enter the verification code and then click on the **Verify** button to continue.  You will be directed to the main Salesforce page.

<a name="token"></a>
## Set and Retrieve the API Security Token

In order to access the Salesforce API's an application must provide a Security Token (or "API Key") when invoking the API.  If you have just created a new Salesforce account, or if you don't have the Security Token for an existing Salesforce account you may generate a new one by perform the following steps.

{% include note.html content="Keep in mind that when you change the Security Token for your Salesforce account all other applications that access Salesforce API's using that account will need to be updated as well." %}
 
1. Navigate to the [Salesforce.com](https://login.salesforce.com) website and login using your developer account.  Once you have logged in open the drop down list next to your username and select the **Setup** link.

	![](./images/pots/mq-sfdc/salesforce_apikey_1.png)

2. Expand the **Personal** menu item located on the left of the page, click on the **Reset My Security Token** menu item and then click on the **Reset Security Token** button. 

	![](./images/pots/mq-sfdc/salesforce_apikey_2.png)

	
3. The new Security Token will be emailed to you. Please access this email and make a note of the security token.  

{% include note.html content="Remember that every time you change your password, the security token will be changed as well. It is important note that when you copy and paste the token into the endpoint configuration that you trim out any leading or trailing spaces as that might cause an authentication error." %}


	
