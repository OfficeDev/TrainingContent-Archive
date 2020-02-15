# Add Authentication to a Bot

In this demo, you will demonstrate the construction of a bot dialog that acquires an access token from Azure Active Directory and calls the Microsoft Graph API.

## Application Registration worksheet

This demo requires the registration of multiple applications in Azure Active Directory (Azure AD), the Bot Framework and the Azure Portal. The **LabFiles** folder of this module contains a file named **AppWorksheet.txt** which can be used to record the various ids and secrets generated and used in the demo.

## Prerequisites

Developing apps for Microsoft Teams requires preparation for both the Office 365 tenant and the development workstation.

For the Office 365 Tenant, the setup steps are detailed on the [Prepare your Office 365 Tenant page](https://docs.microsoft.com/en-us/microsoftteams/platform/get-started/get-started-tenant). Note that while the getting started page indicates that the Public Developer Preview is optional, this lab includes steps that are not possible unless the preview is enabled. Information about the Developer Preview program and participation instructions are detailed on the [What is the Developer Preview for Microsoft Teams? page](https://docs.microsoft.com/en-us/microsoftteams/platform/resources/dev-preview/developer-preview-intro).

### Administrator credentials to tenant

This lab requires delegated permissions that are consented by a tenant administrator. If you are not an administrator in your tenant, you can request a developer tenant through the [Office 365 developer program](https://developer.microsoft.com/en-us/office/dev-program)

### Azure Subscription

The Azure Bot service requires an Azure subscription. A free trial subscription is sufficient.

### Download ngrok

As Microsoft Teams is an entirely cloud-based product, it requires all services it accesses to be available from the cloud using HTTPS endpoints. To enable the exercises to work within Microsoft Teams, a tunneling application is required.

This lab uses [ngrok](https://ngrok.com) for tunneling publicly-available HTTPS endpoints to a web server running locally on the developer workstation. ngrok is a single-file download that is run from a console.

## Update Demo solution

Make the following updates to the demo solution.

1. Launch **Visual Studio 2017** as an administrator: right-click **Visual Studio 2017** and select **Run as administrator**.

1. In Visual Studio 2017, select **File > Open > Project/Solution**.

1. Select the **officedev-talent-management.sln** file from the **Demos\02-add-authentication-bot\solution** folder.

### Find the project URL

1. In **Solution Explorer**, double-click on **Properties**.

1. In the **Properties** designer, select the **Web** tab.

1. Note the **Project URL**.

    ![Screenshot of Solution Explorer highlighting project URL.](../../Images/Demo-01.png)

### Run the ngrok secure tunnel application

1. Open a new **Command Prompt** window.

1. Change to the directory that contains the **ngrok.exe** application.

1. Run the command `ngrok http [port] -host-header=localhost:[port]`. Replace `[port]` with the port portion of the URL noted above.

1. The ngrok application will fill the entire prompt window.

    > **NOTE:** Record the **Forwarding address** using https on the AppWorksheet as the "ngrok forwarding address".

1. Minimize the ngrok command prompt window. It is no longer referenced in this lab, but it must remain running.

    ![Screenshot of ngrok command prompt with local host highlighted.](../../Images/Demo-02.png)

## Register application in the Azure Active Directory

1. Open the [Azure Active Directory admin center](https://aad.portal.azure.com).

1. Log in with the work or school account that is an administrator in the tenant.

1. Select **Azure Active Directory** in the left-most blade.

1. Select **App registrations** in the left-hand menu.

1. Select **New registration**.

1. Enter a name for the application. A suggested name is `Talent Management application` which distinguishes this application from the bot. Select **Register**.

1. In the **Overview** blade, copy the **Application (client) ID**.

    > **NOTE:** Record the **Application (client) ID** on the AppWorksheet as the **AzureAppID**.

1. In the **Overview** blade, , copy the **Directory (tenant) ID**.

    > **NOTE:** Record the **Directory (tenant) ID** on the AppWorksheet as the **AzureTenantID**.

1. Select **Authentication** in the left-hand menu.

1. In the **Redirect URIs section, enter the following address for the **Redirect URI**: `https://token.botframework.com/.auth/web/redirect`. Leave the **Type** as **Web**.

1. Select **Save** from the toolbar at the top of the Authentication blade.

1. Select **Certificates & secrets** in the left-hand menu.

1. Select **New client secret**. Enter a description and select an expiration. Select **Add**.

1. Record the client secret value.

    > **NOTE:** Record the client secret value on the AppWorksheet as the **AzureAppSecret**.

1. Select **API permissions** in the left-had menu.

1. In the **API permissions** blade, select **Add a permission**. Select **Microsoft Graph**. Select **Delegated permissions**.

1. The following permissions are required for the lab. Select any that are not included by default:

- openid (Sign users in)
- profile (View users' basic profile)
- Group > Group.Read.All (Read all groups)
- User > User.Read (Sign in and read user profile)
- User > User.Read.All (Read all users' full profiles)

Select **Add permissions**.

Select **Grant admin consent for [Directory]**. Select **Yes** in the confirmation banner.

  ![Screenshot of Azure Active Directory portal with the requested permissions displayed.](../../Images/Exercise1-01.png)

## Create a Bot Service Channel registration

1. Close all open blades in the Azure Portal.

1. Select **Create a resource**.

1. In the **Search the marketplace** box, enter `bot`.

1. Choose **Bot Channels Registration**

1. Select the **Create** button.

1. Complete the **Bot Channels Registration** blade. For the **Bot name**, enter a descriptive name.

1. Enter the following address for the **Messaging endpoint**. Replace the token `[from-ngrok]` with the value record on the AppWorksheet as **ngrok forwarding address id**.

    ```
    https://[from-ngrok].ngrok.io/api/Messages
    ```

1. Allow the service to auto-create an application.

1. Select **Create**.

1. When the deployment completes, navigate to the resource in the Azure portal. In the left-most navigation, select **All resources**. In the **All resources** blade, select the Bot Channels Registration.

    ![Screenshot of bot channel registration.](../../Images/Starter-03.png)

1. In the **Bot Management** section, select **Channels**.

    ![Screenshot of channel menu with Microsoft Teams icon highlighted.](../../Images/Starter-04.png)

1. Click on the Microsoft Teams logo to create a connection to Teams. Select **Save**. Agree to the Terms of Service.

    ![Screenshot of MSTeams bot confirmation page.](../../Images/Starter-05.png)

1. In the **Bot Management** section, select *Settings**.

1. Select **Add Setting** in the **OAuth Connection Settings** section.

    - For Name, enter `TalentManagementApplication`.
      > NOTE: Record this name on the AppWorksheet as "OAuthConnectionName".
    - For Service Provider, select `Azure Active Directory V2`. Once you select this, the Azure AD-specific fields will be displayed.
    - For **Client id**, enter the value recorded on the AppWorksheet as **AzureAppID**.
    - For **Client secret**, enter the value recorded on the AppWorksheet as **AzureAppSecret**
    - For **Tenant ID**, enter the value recorded on the AppWorksheet as **AzureTenantID**.
    - For **Scopes**, enter `https://graph.microsoft.com/.default`

1. Select Save.

### Record the Bot Channel Registration Bot Id and secret

The Visual Studio solution will use the Bot Channel Registration, replacing the Bot Framework registration.

1. In the **Bot Channels Registration** blade, select **Settings** under **Bot Management**

1. The **Microsoft App Id** is displayed. Record this value.

    > NOTE: Record the **Microsoft App Id** on the AppWorksheet as the "BotChannelRegistrationId".

1. Next to the **Microsoft App Id**, select the **Manage** link. This will open the Application Registration Portal in a new tab. If prompted, log in with the same credentials used for the Azure Portal.

1. In the **Application Secrets** section, select **Generate New Password**. A new password is created and displayed in a popup dialog. Record the new password.

    > NOTE: Record the password on the AppWorksheet as the "BotChannelRegistrationPassword".

1. You may close the browser tab containing the Application Registration Portal. It is no longer needed.

## Configure the web project

The bot project must be configured with information from the registration.

1. In **Visual Studio**, open the **Web.config** file. Locate the `<appSettings>` section.

1. Replace the token **[MicrosoftAppId]** with the value from the AppWorksheet named **BotChannelRegistrationId**.

1. Replace the token **[MicrosoftAppPassword]** with the value from the AppWorksheet named **BotChannelRegistrationPassword**.

1. Replace the token **[OAuthConnectionName]** with the value from the AppWorksheet named **OAuthConnectionName**.

1. Save and close the **web.config** file.

1. Open the **manifest.json** file just added to the project. The `manifest.json` file requires several updates:
    - The `id` property must contain the app ID from registration. Replace the token `[microsoft-app-id]` with the value from the AppWorksheet named **BotChannelRegistrationId**.
    - The `packageName` property must contain a unique identifier. The industry standard is to use the bot's URL in reverse format. Replace the token `[from-ngrok]` with the value from the AppWorksheet named **ngrok forwarding address id**.
    - The `developer` property has three URLs that should match the hostname of the Messaging endpoint. Replace the token `[from-ngrok]` with the unique identifier from the forwarding address.
    - The `botId` property in the `bots` collection property also requires the app ID from registration. Replace the token `[microsoft-app-id]` with the value from the AppWorksheet named **BotChannelRegistrationId**.
    - Save and close the **manifest.json** file.

1. Press **F5** to build the solution and package and start the web service in the debugger. The debugger will start the default browser, which can be ignored. The next step uses the teams client.

### Upload app into Microsoft Teams

Although not strictly necessary, in this lab the bot will be added to a new team.

1. In the Microsoft Teams application, click the **Add team** link. Then click the **Create team** button.

    ![Screenshot of Microsoft Teams with Create Team button highlighted.](../../Images/Demo-07.png)

1. Enter a team name and description. Select **Next**.

1. Invite others from the organization to the team. The demo provides more impact with members in addition to the owner.

1. The new team is shown. In the left-side panel, select the ellipses next to the team name. Choose **Manage team** from the context menu.

    ![Screenshot of Microsoft Teams with Manage Team highlighted.](../../Images/Demo-08.png)

1. On the Manage team display, select **Apps** in the tab strip. Then select the **Upload a custom app** link at the bottom right corner of the application.

1. Select the zip file from the **bin** folder that represents your app. Select **Open**.

1. The app is loaded to the Team. The description and icon for the app is displayed.

    ![Screenshot of Microsoft Teams with new app displayed.](../../Images/Demo-09.png)

### Use the new Profile command

1. In a channel conversation, "at" mention the bot and issue the command `profile`.

1. The bot will attempt to acquire a token for the current user from the Azure Bot Service. If the token is stale, missing, does not have the requested scopes or is otherwise not valid, the bot will reply with a sign-in card.

    ![Screenshot of bot with signin card](../../Images/Exercise2-01.png)

1. On the sign-in card, select **Sign in**. Microsoft Teams will display a popup dialog with the Azure AD login. Complete the login.

1. Once sign-in ins complete, the bot will access profile information for the current user and write a message.

    ![Screenshot of bot with profile information message](../../Images/Exercise2-02.png)
