# Installing and configuring the Nutanix Calm plugin for ServiceNow

Nutanix Calm is the self-service portal product of Nutanix. With Nutanix Calm it is possible for enterprises to automate infrastructure and application deployments.
In this document we will show how to install and configure the Nutanix Calm plugin for ServiceNow. ServiceNow is a tool that is widely used by many enterprises. Integrating ServiceNow with Nutanix Calm offers end-users the possibility to create, manage and delete Nutanix Calm applications directly via the ServiceNow web interface.

# General overview

If you want to install the Nutanix Calm plugin for ServiceNow, you need to have admin access to a ServiceNow instance. If you don't have a ServiceNow instance available, but you want to try the Nutanix Calm integration, it is possible to request a temporary developer instance via this link: 
https://developer.servicenow.com/dev.do 

**Note**: Version 1.2 of the plugin will be installed on a `New York` version of ServiceNow

# Install and configure the Nutanix Calm plugin
ServiceNow is a product that runs in the cloud. A `MID` server needs to be installed in order to allow ServiceNow to communicate with Nutanix Calm. In this example I use a Windows 2016 server that is joined to an Active Directory domain. 
Below you can see a high-level overview of the solution.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/setup.png?raw=true)


## Installing the mid server

To install the `MID` server, first login on your ServiceNow instance and search for the `MID` server section. Go to Downloads and select the correct version of the installation files. In this case it is the `Windows 64 bit` variant. Copy the installation file to your Windows 2016 server.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/0_download_mid_server.png?raw=true)

Create a directory on your Windows server and copy the installation files to the newly created folder. Extract the zip file and run the `installer.bat` file in the `agent directory`.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/1_extract_mid_installer.png?raw=true)

The `installer.bat` file will open an installation wizard. Enter your ServiceNow instance URL and credentials. Test your connection before proceeding clicking `next`. 

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/2_install_mid_1.png?raw=true)

On the next screen give your `MID` server a name and press `next`.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/3_name_mid_server.png?raw=true)

Press `next` after verifying the configuration.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/4_verify_pre_install.png?raw=true)

Start your `MID` server and press `Exit`.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/5_start_mid.png?raw=true)

If you want to double check if the service is running, open `services.msc` and make sure the `ServiceNow MID Server_<MID server name>` service is running.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/6_check_service.png?raw=true)

When the service is running, go back to the ServiceNow web interface and go to the `Servers` board in the `MID` server section and check if the newly installed `MID` server is listed. You will see the server is not yet validated. Click the checkbox next to your `MID` server entry and run the `validate` action. 

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/7_validate_mid.png?raw=true)

Check if all fields in the pop-up box are enabled and press save. Now the `MID` server will be validated. This can take a few minutes to complete.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/8_confirm_validation.png?raw=true)

Do not proceed to the next steps before the `MID` server is validated. 

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/9_validated_mid.png?raw=true)


## Plugin pre-requisites

Additional pre-requisites must be checked before we can install the plugin. 

### Global application scope
Make sure you are in the global application scope. This can be verified by going to the `settings` in the right upper corner, go to the `Developer` tab and make sure the value for `Application` is set to `Global`. Optionally you can enable `Show application picker in header`. This will show the current application scope at the right upper corner of the ServiceNow web interface.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/10_globalapplication.png?raw=true)


### Table permissions
Verify if all of the tables listed below have the required permissions.

| Table               | Label                | Permissions         |
| --------------------|:---------------------|:--------------------|
|sys_user_has_role    |User Role             |Read, Create, Update |
|sys_user_grmember    |Group Member          |Read, Create, Update |
|sys_group_has_role   |Group Role            |Read, Create, Update |
|item_option_new      |Variable              |Read, Create, Update |
|sys_user_group       |Group                 |Read, Create, Update |
|sc_category          |Category              |Read, Create, Update |
|sc_catalog           |Catalog               |Read, Create, Update |
|catalog_ui_policy    |Catalog UI Policy     |Read, Create, Update |
|catalog_script_client|Catalog Client Scripts|Read, Create, Update |
|user_criteria        |User Criteria         |Read, Create, Update |
|question             |Question              |Read, Create, Update |
|question_choice      |Question Choice       |Read, Create, Update |

You can verify the permissions by checking the `Tables` dashboard in the `System Definition` section. You can use the filter at the top of the page to find the correct table.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/11_search_def_table.png?raw=true)

Click on the correct table, go to `Application Access` and check the permissions. When the permissions are correct, click `Update`.
Repeat this procedure for all the tables.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/12_check_app_permissions.png?raw=true)

### Additional system properties

Enter `sys_properties.LIST` in the search bar on the left of the ServiceNow web interface. This will open a different tab in your browser. Search for the `glide.sc.guide.tab.validate` key and set the value to `true` and press `Update`.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/13_sys_props_list.png?raw=true)
![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/14_search_sys_prop.png?raw=true)
![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/15_mod_sys_prop.png?raw=true)

### Install the User Criteria Scoped API Plugin

Go to the `Plugin` section, search for the plugin using ID `com.glideapp.user_criteria.scoped.api` and install it.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/16_install_user_scoped_plugin.png?raw=true)

## Installing and configuring the plugin
There are two methods of installing the Nutanix Calm ServiceNow plugin. The first method is a manual installation, the second one is via the ServiceNow Store. Both methods are shown in this document.

### Installing v1.2 plugin via ServiceNow Store
On the ServiceNow web interface, enter `System Applications` in the filter field on the left side of the screen. Click on the `All` section under the `All Available Applications` header and enter `Calm` in the search bar. The plugin will be ready for configuration after clicking `Install`.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/35_search_plugin.png?raw=true)

If the application is not found, login on the ServiceNow store and allow your ServiceNow instance to install the Calm plugin. Follow the steps in this video to add the application to your ServiceNow instance:

https://www.youtube.com/watch?v=JlHc-nA3e6Y

Contact your ServiceNow administrator if you do not have a login for the ServiceNow store. 

**Note**: It is not allowed to install applications on ServiceNow developer instances using the ServiceNow Store. Please follow the manual procedure to install the Nutanix Calm ServiceNow plugin.



### Installing v1.1 and v1.2 plugin manually

Download or git clone the files in following git repo: 
https://github.com/nutanix/Calm-Servicenow-Plugin

You will see several folders in this repo. We will require both the v1.1 and v1.2 folder. 
Initially we will use the `Nutanix Full Certified Build(v1.0+v1.1).xml` file in the v1.1 folder. 

Go to the `Retrieved Update Sets` section and upload the `Nutanix Full Certified Build(v1.0+v1.1).xml` file

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/17_upload_1.1_xml.png?raw=true)
![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/18_upload_1.1_final_xml.png?raw=true)

When the file is uploaded, go into the `Nutanix Calm` update set.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/19_uploaded_set.png?raw=true)

Press `Preview Update Set` and wait until completion. If you get an error on this step, verify the pre-requisites earlier in this document.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/20_preview_update_set.png?raw=true)

Press `Commit Update Set` when the preview process completes.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/21_commit_update_set.png?raw=true)

Perform the same procedure for the `Nutanix Calm V1.2_5.xml` file in the v1.2 folder.

Below you can see the result of uploading the two XML files and performing a preview and a commit on both.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/22_uploaded_both.png?raw=true)

#### Upgrading v1.1 to v1.2 manually

The process for upgrading the v1.1 version of the plugin to v1.2 is analogous to the process defined in the `Installing v1.1 and v1.2 plugin manually` section.

Go to the `Retrieved Update Sets` section and upload the `Nutanix Calm V1.2_5.xml` file in the `Calm-Servicenow-Plugin` git repo and commit the update set. 
**Note**: Check the `Installing v1.1 and v1.2 plugin manually` section for the detailed steps.

After uploading and commiting the update set, you are now using v1.2 of the plugin.

### Configuring the plugin

After completing the steps above, we can now configure the Nutanix Calm plugin.
In the search bar on the left, search for `calm`. 

**Note**: You can press the star icon next to the Nutanix Calm section to create a favorite.

It is important to change your application profile from `Global` to `Nutanix Calm` at this moment in time. 

Go to `Application Properties` and fill in the required information:

- Search for your `MID` server by pressing the search icon next to the input field
- Press the lock icon next to the `Calm Instance` field and enter the URL to your Calm instance in following format: `https://<hostname>:<port>`
- Enter your Calm username and password
- Configure your `Approval workflow`. If you want to auto-approve, choose `Nutanix - Auto Approve` by pressing the search icon. 
- Configure the `Assignment Group`. In this case I selected a random one.

When everything is entered, press `Save Properties` 
A connection and login attempt to your Calm instance will be performed. If everything was successful, a `Sync Now` button will become visible. Click it before continuing.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/23_configure_calm.png?raw=true)

The duration of the syncing process depends on the size of your Calm environment. The sync is complete when the `Sync is in progress` bar on the `Application Properties` section disappears.

You can now browse through your projects, blueprints and deployed applications via ServiceNow. 


## Create a catalog item

If you want to provision an application via ServiceNow, you first need to create a `Catalog item` for the blueprint. 
Go to `Catalog Items` and press `New`.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/24_new_catalog_item.png?raw=true)

Choose your project, blueprint and application profile for your catalog item.
![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/25_create_catalog_item_1.png?raw=true)

On the `Choose Options` tab you can check all the input parameters defined on your blueprint.
![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/26_create_catalog_item_2.png?raw=true)
On the `Checkout` tab you give a name and a description to your catalog item. When you are ready, press `Checkout`.
![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/27_create_catalog_item_3.png?raw=true)

The catalog item is now being created and can be requested after a few minutes. Go back to the `Catalog Items` section and check if the item is present.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/28_catalog_item_created.png?raw=true)

### Launch catalog item

To launch the catalog item, click on it and press `Launch`.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/29_launch_1.png?raw=true)

Enter the mandatory information and press `Order Now`.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/30_launch_2.png?raw=true)

This will trigger the required API calls via the `MID` server to your Calm instance. You can check the progress by checking the `Orders` section, or by opening the Calm UI.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/31_running_app.png?raw=true)
![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/32_orders.png?raw=true)

When the order is complete, you can find it in the `Applications` section.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/33_app.png?raw=true)

### Performing day-2 operations

If you want to perform an action on a deployed application, go to `Applications`, choose your application and scroll down. 

Pick the correct action (in this case we will restart the Windows server), right(!) click and choose `Perform Action`. 

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/34_day2.png?raw=true)

The action is now invoked.

![alt text](https://github.com/yannickstruyf3/calm-servicenow-install/blob/master/images/34_invoked_day2.png?raw=true)


# Summary
This document demonstrated how to:
- Install a `MID` server
- Install and configure the Nutanix Calm ServiceNow plugin
- Create a catalog item
- Launch a catalog item
- Perform a day-2 operation on a provisioned application

This document can also be found on the Nutanix Belux blog: http://thecloudsaloon.com/installing-and-configuring-the-nutanix-calm-plugin-for-servicenow/