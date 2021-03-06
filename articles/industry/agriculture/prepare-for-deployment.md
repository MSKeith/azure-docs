---
title: Deploy Azure FarmBeats
description: Describes how to deploy FarmBeats
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-umha
---

# Deploy FarmBeats

This article describes how to set up Azure FarmBeats.

Azure FarmBeats is an industry-specific, extensible solution for data-driven farming that enables seamless provisioning and sensor devices connectivity to Azure cloud, telemetry data collection, and aggregation. Azure FarmBeats has various sensors such as cameras, drones, soil sensors, and management of devices from the cloud, which includes infrastructure and services in Azure for the IoT-ready (Internet of Things) devices to an extendible web and mobile app to provide visualization, alerts, and insights.

> [!NOTE]
> Azure FarmBeats is supported only in Public Cloud Environments. For more information about cloud environment, see [Azure](https://azure.microsoft.com/overview/what-is-a-public-cloud/).

Azure FarmBeats has the following two components:

- **Data hub** - Data hub is the platform layer of Azure FarmBeats that lets you build, store, process data and draw insights from existing or new data pipelines. This platform layer is useful to run and build your agriculture data pipelines and models.

- **Accelerator** - Accelerator is the solution layer of Azure FarmBeats that has a built-in application to illustrate the capabilities of Azure FarmBeats using the pre-created agriculture models. This solution layer lets you create farm boundaries and draw insights from the agriculture data within the context of the farm boundary.

A quick deployment of Azure FarmBeats should take less than an hour. Costs for the Data hub and Accelerator vary based on usage.

## Deployed resources

Azure FarmBeats deployment creates the below listed resources within your subscription:

|S.No  |Resource Name  |Azure FarmBeats Component  |
|---------|---------|---------|
|1  |       Azure Cosmos DB   |  Data hub       |
|2  |    Application Insights      |     Data hub/Accelerator     |
|3  |Azure Cache for Redis   |Data hub   |
|4  |       Azure KeyVault    |  Data hub/ Accelerator        |
|5  |    Time Series Insights       |     Data hub      |
|6 |      EventHub Namespace    |  Data hub       |
|7  |    Azure Data Factory V2       |     Data hub/ Accelerator      |
|8  |Batch account    |Data hub   |
|9  |       Storage account     |  Data hub/ Accelerator        |
|10  |    Logic app        |     Data hub      |
|11  |    API connection        |     Data hub      |
|12|      App service      |  Data hub/Accelerator       |
|13 |    App service plan        |     Data hub/ Accelerator      |
|14 |Azure Maps account     |Accelerator    |
|15 |       Time Series Insights      |  Data hub     |

Azure FarmBeats is available for you to download from the Azure Marketplace. You can access it directly from Azure portal.  

## Prepare

You need the following permissions for deploying Azure FarmBeats:

- Tenant: Read Access
- Subscription: contributor to owner
- Resource group: owner

## Before you begin

Before initiating the deployment, ensure you've the following:

- Sentinel account
- Azure Active Directory (app registration)
- Azure FarmBeats

## Create a sentinel account    

An account with sentinel helps you to download the sentinel satellite imagery from their official website to your device. Follow these steps to create a free account:

1. Go to https://scihub.copernicus.eu/dhus/#/self-registration. In the registration page, provide a first name, last name, username, password, and email.
2. In the **Select Domain** drop-down menu, select the option **Land**.
3. In the **Select Usage** drop-down menu, select the option **Education**.
4. In the **Select Country** drop-down menu, select your country.
5. Select **Register** to complete the registration process.

A verification email will be sent to the registered email address for confirmation. Select the link and confirm. Your registration process is complete.

## Create Azure AD app registration

For authentication and authorization on Azure FarmBeats, you must have an Azure active directory application registration which:

- Case 1: Installer can create automatically (provided you have the required tenant, subscription, and resource group access permissions).
- Case 2: You can create and configure before deploying Azure FarmBeats (requires manual steps).

**Case 1**: The installer assumes that you have the rights to create an Azure active directory application registration within the desired subscription. To register, sign in to portal, go to **Azure active directory** > **app registration** > **New registration**.

If you already have a subscription, you can directly moved to the next procedure.

**Case 2**: This method is the preferred step when you don't have enough rights to create and configure an Azure AD app registration within your subscription. Request your  admin to use the [custom script](https://aka.ms/FarmBeatsAADScript), which will help IT admin automatically generate and configure the Azure AD app registration on the Azure portal. As an output to running this custom script using PowerShell environment the IT admin needs to share an Azure Active Directory Application Client ID and password secret with you. Make a note of these values.

Use the following steps to run the Azure AD application registration script:

1. Get the registration [script](https://aka.ms/FarmBeatsAADScript).
2. Sign in to Azure portal and select your subscription and AD tenant.
3. Launch Cloud Shell from the top navigation of the Azure portal.

    ![Project Farm Beats](./media/prepare-for-deployment/navigation-bar-1.png)


4. First-time users will be prompted to select a subscription to create a storage account and Microsoft Azure Files share. Select **Create storage**.
5. First time users will be prompted with a choice of preferred shell experience- Bash or PowerShell. Choose PowerShell.
6. In Cloud Shell enter the below commands to run the script.

    ```powershell
    ./create_aad_script.ps1
    ```
7. Make a note of the Azure AD application ID and client secret to share with person deploying Azure FarmBeats.

## Create Azure FarmBeats offer on marketplace

Use these steps to create an Azure FarmBeats offer in the marketplace:

1. Sign in to the **Azure portal** and select your account in the top-right corner, and switch to the Azure AD tenant where you want to deploy Microsoft Azure FarmBeats.
2. Select **Search/Marketplace** or **Create Resources**. Type **FarmBeats** to view the offer details. Download the guides available via aka.ms hyperlinks at this page before proceeding to the next steps.
3. Select **Create** and enter the following information:

   - Enter subscription name.
   - Enter an existing resource group name (empty resource group only) or create a new resource group for deploying Azure FarmBeats. Make note of this resource group to enter within the input.json file at a later stage.
   - Enter the region you want to install Azure FarmBeats into. FarmBeats can currently be installed in these regions: Central US, West Europe, East US 2, North Europe, West US, Southeast Asia, East US, Australia East, West US 2.
4. Select **OK**, which redirects you to the Terms of use page. Review the standard marketplace terms or select the hyperlink to review the Terms of Use.
5. Select **Close**, then the "I agree" checkbox and then select **Create**.
6. You have now successfully signed Azure FarmBeats's End-user License agreement (EULA) on the marketplace. To continue with the deployment, follow the next steps in this guide.

### Prepare Input.Json file

This file is your input file to Azure Cloud Shell and parameters whose values are specified within this file before uploading aren't prompted during deployment, therefore saving you some time.  

> [!NOTE]
> This file inputs values to Azure Cloud Shell.  To save time, during deployment you won’t be prompted for parameters you add to this file. You will be prompted for missed parameters.

Review the parameters before preparing the file.

|Command | Description|
|--- | ---|
|“sku”  | Provides a choice to download either or both the components of Azure FarmBeats. Specifies which components to download. To install only Data hub, use “onlydatabhub”. To install Data hub and Accelerator, use “both”.|
|“subscriptionId”  | Specifies the subscription for installing FarmBeats|
|“datahubResourceGroup”  | Resource group name for Data hub resources.|
|“datahubLocation” | Location where you would like to store the Data hub resources.|
|“acceleratorWebsiteName”  |Unique URL prefix to name your Data hub
Swagger website. The default is Data hub resource group name. Press enter to continue with the default value.|
|“acceleratorResourceGroup”  | Resource group name for Data hub resources. |
|“acceleratorLocation”  | Location for storing the Data hub resources
“acceleratorWebsiteName”  | Unique URL prefix to name your Accelerator website. The default is accelerator. Press enter to continue with the default value.|
|''sentinelUsername'' | user name to sign into: https://scihub.copernicus.eu/dhus/#/self-registration.|
|“sentinelPassword”  | Password to sign into: https://scihub.copernicus.eu/dhus/#/self-registration.|
|“farmbeatsAppId"  | Values to be shared by Team Azure FarmBeats.  |
|"farmbeatsPassword"  | Values to be shared by Team Azure FarmBeats.|
|“notificationEmailAddress”  | Email address to receive the notifications for any alerts that you configure within Data hub.|
|“upda"aadAppClientId" "  |[**Optional**] Parameter to be included within Input.Json only if Azure AD app already exists.  - True/False. False for a first-time installation and True for Upgrade scenario.|
|"aadAppClientId"  | [**Optional**] Parameter to be included within Input.Json only if Azure AD app already exists.  |
|"aadAppClientSecret"   | [**Optional**] Parameter to be included within Input.Json only if Azure AD app already exists.|


Sample JSON input:

```json
{  
   "sku":"both", 
   "subscriptionId": "da9xxxec-dxxf-4xxc-xxx21-xxx3ee7xxxxx", 
   "datahubResourceGroup": "dummy-test-dh1", 
   "datahubLocation": "westus2", 
   "datahubWebsiteName": "dummy-test-dh1", 
   "acceleratorResourceGroup": "dummy-test-acc1", 
   "acceleratorLocation": "westus2", 
   "acceleratorWebsiteName": "dummy-test-acc1", 
   "sentinelUsername": "dummy-dev", 
   "farmbeatsAppId": "c3cb3xxx-27xx-4xxb-8xx6-3xxx2xxdxxx5c", 
   "notificationEmailAddress": "dummy@microsoft.com", 
   "updateIfExists": true
}
```
## Deploy within Cloud Shell browser-based command line

As part of the marketplace workflow you've created one Resource Group and signed the End-user License Agreement, which can be reviewed once again as part of the actual deployment. The deployment will be done via Azure Cloud Shell (browser-based command line) using Bash environment.  

> [!NOTE]
> Inactive Cloud Shell sessions expire after 20 minutes. Try to complete the deployment within this time.

1. Sign into **Azure** portal and select the desired subscription and AD tenant.
2. Launch **Cloud Shell** from the top navigation of the **Azure** portal.

   ![Project Farm Beats](./media/prepare-for-deployment/navigation-bar-1.png)

3. Select a subscription to create a storage account and Microsoft Azure Files share.
4. Select **Create storage**.
5. Select the environment drop-down from the left side of shell window (Bash).

   ![Project Farm Beats](./media/prepare-for-deployment/bash-1-1.png)

### Deployment scenario 1

Installer creates the Azure AD (you have AD tenant read permissions).  

1. Download the [input.json](https://aka.ms/PPInputJsonTemplate) template. Include the Azure Application client ID and password in the input.json file, and save it.
2. Open the downloaded file in a notepad and populate the file by entering the values.

    ```json
    {
    "sku":"both",
    "subscriptionId":"daxx9xxx-d18f-4xxc-9c21-5xx3exxxxx45",
    "datahubResourceGroup":"dummy-test-dh1",  
    "location":"westus2",  
    "datahubWebsiteName":"dummy-test-dh1",  
    "acceleratorResourceGroup":"dummy-test-acc1",  
    "acceleratorWebsiteName":"dummy-test-acc1",  
    "sentinelUsername":"dummy-dev",
    "notificationEmailAddress":"dummyuser@org1.com",
    "updateIfExists":true
    }

    ```

    > [!NOTE]
    > You can skip this step and the inputs will prompt at run-time.

3. Save the file and make a note of the path (on your local computer).  
4. Go to Azure Cloud Shell and after successful authentication, select the upload (see highlighted icon in below image) and upload the input.json file to Cloud Shell storage. You need not pass the Azure AD parameter within the json as installer will be creating and configuring the Azure AD app registration for you.

   ![Project Farm Beats](./media/prepare-for-deployment/bash-2-1.png)

5. Type or paste the “Deployment Command” into the Cloud Shell. Make sure to modify the path to input. Json file and press enter.  

    ```
    wget -N -O farmbeats-installer.sh https://aka.ms/FB_1.2.0 && bash farmbeats-installer.sh> /home/dummyuser/input.json

    ```
   The script automatically downloads all dependencies, and builds the deployer. Then, you're prompted to agree to the Azure FarmBeats End-user license agreement (EULA).

   - Enter ‘Y’ if you agree and you will proceed to the next step.
   - Enter ‘N’ if you do not agree to the terms and the deployment will terminate.

   Then you will be prompted to enter an access token for the deployment. Copy the code generated and login to https://microsoft.com/devicelogin with your Azure credentials.

   > [!NOTE]
   > The code expires after 60 minutes. When it expires you can restart by typing the deployment command again.

6. On the next prompt, enter sentinel account password.
7. The installer now validates and starts deploying, which can take about 20 minutes.
8. Once the deployment is successful, you will receive the below output links:
    - **Data hub URL**: Swagger link to try Azure FarmBeats APIs.
    - **Accelerator URL**: User interface to explore Azure FarmBeats Smart Farm Accelerator.
    - **Deployer log file**- Log file created during deployment. It can be used for troubleshooting.

    - Enter 'Y' if you agree and you'll continue to the next step.
    - Enter 'N' if you don't agree to the terms and the deployment will terminate.

   Then you'll be prompted to enter an access token for the deployment. Copy the code generated and sign in to https://microsoft.com/devicelogin with your Azure credentials.

   > [!NOTE]
    > Make note of these and keep the deployment log file path handy for future use.


### Deployment scenario 2

Existing Azure Active Directory app registration is used to deploy.

1. Download [input json](https://aka.ms/PPInputJsonTemplate) Include the Azure Application Client ID and password in the input.json, save it.

    ```json
       {
       "sku":"both",
       "subscriptionId":"daxx9xxx-d18f-4xxc-9c21-5xx3exxxxx45",
       "datahubResourceGroup":"dummy-test-dh1",  
       "location":"westus2",  
       "datahubWebsiteName":"dummy-test-dh1",  
       "acceleratorResourceGroup":"dummy-test-acc1",  
       "acceleratorWebsiteName":"dummy-test-acc1",  
       "sentinelUsername":"dummy-dev",
       "notificationEmailAddress":"dummyuser@org1.com",
       "updateIfExists": true,
       "aadAppClientId": "lmtlemlemylmylkmerkywmkm823",
       "aadAppClientSecret": "Kxxxqxxxxu*kxcxxx3Yxxu5xx/db[xxx"
       }
     ```

     Follow the rest of the steps:

2. Make a note of the path to your input.json file (on your local computer).
3. Go to Azure Cloud Shell once again and you're successfully authenticated, select the upload button (see highlighted icon in below image) and upload the input.json file to Cloud Shell storage.

    ![Project Farm Beats](./media/prepare-for-deployment/bash-2-1.png)

4. Type or paste the *Deployment Command* into the Cloud Shell. Make sure to modify the path to input. Json file and press enter.

    ```
    wget -N -O farmbeats-installer.sh https://aka.ms/FB_1.2.0 && bash farmbeats-installer.sh> /home/dummyuser/input.json

    ```
   Follow the rest of the steps:

5. The script automatically downloads all dependencies, and builds the deployer.
6. You'll be prompted to read and agree to the Azure FarmBeats End-user license agreement (EULA).

   - Enter 'Y' if you agree and you'll continue to the next step.
   - Enter 'N' if you don't agree to the terms and the deployment will terminate.

7. You'll be prompted to enter an access token for the deployment. Copy the code generated and sign in to https://microsoft.com/devicelogin with your Azure credentials.
8. The installer will now validate and start creating the resources, which can take about 20 minutes. Keep the session active on Cloud Shell during this time.
9. Once the deployment goes through successfully, you'll receive the below output links:
   - **Data hub URL**: Swagger link to try FarmBeats APIs.
   - **Accelerator URL**: User interface to explore FarmBeats Smart Farm Accelerator.
   - **Deployer log file**: saves logs during deployment and can be used for troubleshooting.

If you encounter any issues, review [Troubleshoot](troubleshoot-project-farmbeats.md).


## Validate deployment

### Data hub

Once the data hub installation is complete, you'll receive the URL to access Azure FarmBeats APIs via the Swagger interface in the format: https://\<yourdatahub-website-name>.azurewebsites.net

1. To sign in via Swagger, copy and paste the URL in the browser.
2. Sign in with Azure portal credentials.
3. Sanity test (Optional)

     - Able to successfully sign in to the Swagger portal using the Data hub link, which you received as an output to a successful deployment.
     - Extended types Get API- Select "Try it out /Execute"
     - You should receive the server response Code 200 and not an exception such as 403 "unauthorized user".

### Accelerator

Once the Accelerator installation is complete, you'll receive the URL to access FarmBeats user-interface in the format: https://\<accelerator-website-name>.azurewebsites.net

1. To sign in from Accelerator, copy and paste the URL in the browser.
2. Sign in with Azure portal credentials.
3. Run an optional sanity test.

    - Check if you are able to successfully sign in to the Accelerator portal using the Accelerator link that you received as an output to a successful deployment.
    - Select **Create farm**.
    - Under the icon "?" open the FarmBeats guides using the **Get started** button.

## Upgrade

The steps for upgrade are similar to the first-time installation. Follow these steps:

1. Sign in to Azure portal and select your desired subscription and AD tenant.
2. Launch Cloud Shell from the top navigation of the Azure portal.

   ![Project Farm Beats](./media/prepare-for-deployment/navigation-bar-1.png)

3. Select a subscription to create a storage account, and an Azure Files file share.
4. Select **Create storage**.
5. Select the environment from the drop-down from the left of the of shell (Bash).
6. Make changes to your input.json file only if needed and upload to the Azure Cloud Shell. For example, you can update your email address for the notification you want to receive.
7. Make note of the input.json file path (on your local computer).
8. Go to Azure Cloud Shell. Once you're successfully authenticated, select the upload button (see highlighted icon in below image) and upload the input.json file to Cloud Shell storage.

   ![Project Farm Beats](./media/prepare-for-deployment/bash-2-1.png)

9. Type or paste the **Deployment Command** into the Cloud Shell. Make sure to modify the path to input. json file and press enter.

    ```
    wget -N -O farmbeats-installer.sh https://aka.ms/FB_1.2.0 && bash farmbeats-installer.sh> /home/dummyuser/input.json

    ```
    Follow the rest of the steps:

10. The Installer automatically prompts the required inputs on run-time:

    - Enter an access token for deployment. Copy the code generated and sign in to
     https://microsoft.com/devicelogin with your Azure credentials.

     > [!NOTE]
     > The code expires after 60 minutes. When it expires you can restart by typing the deployment command again.

     - Sentinel password

11. The installer now validates and starts creating the resources, which can take about 20 minutes.
12. Once the deployment is successful, you will receive the below output links:

    - **Data hub URL**: Swagger link to try FarmBeats APIs.
    - **Accelerator URL**: User interface to explore FarmBeats Smart Farm Accelerator.
    - **Deployer log file**:  saves logs during deployment. It can be used for troubleshooting.

    > [!NOTE]
    > Make note of the above links and keep the deployment log file path handy for future use.

## Uninstall

Currently we don't support automated uninstallation of FarmBeats using the installer. To remove the Data hub or  Accelerator, in the Azure portal, delete the resource group in which these components are installed, or delete resources manually.

For example, if you deployed Data hub and Accelerator in two different resource groups, you delete those resource groups as follows:

1. Sign into the Azure portal.
2. Select your account in the top right corner, and switch to the desired Azure AD tenant where you want to deploy Microsoft FarmBeats.

   > [!NOTE]
   > Data hub is needed for Accelerator to work properly. We don’t recommend uninstalling Data hub without uninstalling Accelerator.

3. Select Resource Groups, and type in the name of the Data hub or Accelerator resource group that you want to delete.
4. Select the resource group name. Type in the name again to double-check, and click Delete to remove the resource group, and all its underlying resources.
5. Alternatively, you can delete each resource manually, which is not recommended.
7. To delete/uninstall data hub, go to the Resource group directly on Azure and delete the resource group from there.

## Next steps

You have deployed Azure FarmBeats. Now, learn how to [create farms](manage-farms.md#create-farms).
