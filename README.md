---
services: active-directory
platforms: dotnet
author: dstrockis
---

# Authenticating to Azure AD non-interactively using a username & password

This sample demonstrates a .Net console application calling a web API that is secured using Azure AD. The .Net application uses the Active Directory Authentication Library (ADAL) to obtain a JWT access token through the OAuth 2.0 protocol. The access token is sent to the web API to authenticate the user. This sample shows you how to use ADAL to authenticate users via raw credentials (username and password, or Windows integrated authentication) via a text-only interface.

For more information about how the protocols work in this scenario and other scenarios, see [Authentication Scenarios for Azure AD](https://azure.microsoft.com/documentation/articles/active-directory-authentication-scenarios/).

> Looking for previous versions of this code sample? Check out the tags on the [releases](../../releases) GitHub page.

## How To Run This Sample

To run this sample you will need:
- Visual Studio 2013
- An Internet connection
- An Azure subscription (a free trial is sufficient)

Every Azure subscription has an associated Azure Active Directory tenant.  If you don't already have an Azure subscription, you can get a free subscription by signing up at [https://azure.microsoft.com](https://azure.microsoft.com).  All of the Azure AD features used by this sample are available free of charge.

### Step 1:  Clone or download this repository

From your shell or command line:

`git clone https://github.com/Azure-Samples/active-directory-dotnet-native-headless.git`

### Step 2:  Create a user account in your Azure Active Directory tenant

If you already have a user account in your Azure Active Directory tenant, you can skip to the next step.  This sample will not work with a Microsoft account, so if you signed in to the Azure portal with a Microsoft account and have never created a user account in your directory before, you need to do that now.  If you create an account and want to use it to sign-in to the Azure portal, don't forget to add the user account as a co-administrator of your Azure subscription.

### Step 3:  Register the sample with your Azure Active Directory tenant

There are two projects in this sample.  Each needs to be separately registered in your Azure AD tenant.

#### Register the TodoListService web API

1. Sign in to the [Azure management portal](https://manage.windowsazure.com).
2. Click on Active Directory in the left hand nav.
3. Click the directory tenant where you wish to register the sample application.
4. Click the Applications tab.
5. In the drawer, click Add.
6. Click "Add an application my organization is developing".
7. Enter a friendly name for the application, for example "TodoListService", select "Web Application and/or Web API", and click next.
8. For the sign-on URL, enter the base URL for the sample, which is by default `https://localhost:44321`.
9. For the App ID URI, enter `https://<your_tenant_name>/TodoListService`, replacing `<your_tenant_name>` with the name of your Azure AD tenant.  Click OK to complete the registration.
10. While still in the Azure portal, click the Configure tab of your application.
11. Find the Client ID value and copy it aside, you will need this later when configuring your application.

#### Register the TodoListClient app

1. Sign in to the [Azure management portal](https://manage.windowsazure.com).
2. Click on Active Directory in the left hand nav.
3. Click the directory tenant where you wish to register the sample application.
4. Click the Applications tab.
5. In the drawer, click Add.
6. Click "Add an application my organization is developing".
7. Enter a friendly name for the application, for example "TodoListClient-Headless-DotNet", select "Native Client Application", and click next.
8. For the Redirect URI, enter `http://TodoListClient`. Please note that the Redirect URI will not be used in this sample, but it needs to be defined nonetheless. Click finish.
9. Click the Configure tab of the application.
10. Find the Client ID value and copy it aside, you will need this later when configuring your application.
11. In "Permissions to Other Applications", click "Add Application."  Select "Other" in the "Show" dropdown, and click the upper check mark.  Locate & click on the TodoListService, and click the bottom check mark to add the application.  Select "Access TodoListService" from the "Delegated Permissions" dropdown, and save the configuration.

### Step 4:  Configure the sample to use your Azure AD tenant

#### Configure the TodoListService project

1. Open the solution in Visual Studio 2013.
2. Open the `web.config` file.
3. Find the app key `ida:Tenant` and replace the value with your AAD tenant name.
4. Find the app key `ida:Audience` and replace the value with the App ID URI you registered earlier, for example `https://<your_tenant_name>/TodoListService`.

#### Configure the TodoListClient project

1. Open `app.config`.
2. Find the app key `ida:Tenant` and replace the value with your AAD tenant name.
3. Find the app key `ida:ClientId` and replace the value with the Client ID for the TodoListClient from the Azure portal.
4. Find the app key `todo:TodoListResourceId` and replace the value with the  App ID URI of the TodoListService, for example `https://<your_tenant_name>/TodoListService`
5. Find the app key `todo:TodoListBaseAddress` and replace the value with the base address of the TodoListService project.

### Step 5:  Trust the IIS Express SSL certificate

Since the web API is SSL protected, the client of the API (the web app) will refuse the SSL connection to the web API unless it trusts the API's SSL certificate.  Use the following steps in Windows Powershell to trust the IIS Express SSL certificate.  You only need to do this once.  If you fail to do this step, calls to the TodoListService will always throw an unhandled exception where the inner exception message is:

"The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel."

To configure your computer to trust the IIS Express SSL certificate, begin by opening a Windows Powershell command window as Administrator.

Query your personal certificate store to find the thumbprint of the certificate for `CN=localhost`:

```
PS C:\windows\system32> dir Cert:\LocalMachine\My


    Directory: Microsoft.PowerShell.Security\Certificate::LocalMachine\My


Thumbprint                                Subject
----------                                -------
C24798908DA71693C1053F42A462327543B38042  CN=localhost
```

Next, add the certificate to the Trusted Root store:

```
PS C:\windows\system32> $cert = (get-item cert:\LocalMachine\My\C24798908DA71693C1053F42A462327543B38042)
PS C:\windows\system32> $store = (get-item cert:\Localmachine\Root)
PS C:\windows\system32> $store.Open("ReadWrite")
PS C:\windows\system32> $store.Add($cert)
PS C:\windows\system32> $store.Close()
```

You can verify the certificate is in the Trusted Root store by running this command:

`PS C:\windows\system32> dir Cert:\LocalMachine\Root`

### Step 6:  Run the sample

Clean the solution, rebuild the solution, and run it.  You might want to go into the solution properties and set both projects as startup projects, with the service project starting first.

Type the command `add`. You will be prompted for your username and password. Enter them, then enter the description a new todo. That done, type the command `list`. You will see that the command executes without prompting you again. You can clear the token cache by typing `clear`, and leave the client by typing `exit`.
Notice that if you stop the application without clearing the cache, the next time you run the application you won't be prompted to sign-in again - that is the sample implements a persistent cache for ADAL, and remembers the tokens from the previous run.

## How To Deploy This Sample to Azure

To deploy the TodoListService to Azure Web Sites, you will create a web site, publish the TodoListService to the web site, and update the TodoListClient to call the web site instead of IIS Express.

### Create and Publish the TodoListService to an Azure Web Site

1. Sign in to the [Azure management portal](https://manage.windowsazure.com).
2. Click on Web Sites in the left hand nav.
3. Click New in the bottom left hand corner, select Compute --> Web Site --> Quick Create, select the hosting plan and region, and give your web site a name, e.g. todolistservice-contoso.azurewebsites.net.  Click Create Web Site.
4. Once the web site is created, click on it to manage it.  For this set of steps, download the publish profile and save it.  Other deployment mechanisms, such as from source control, can also be used.
5. Switch to Visual Studio and go to the TodoListService project.  Right click on the project in the Solution Explorer and select Publish.  Click Import, and import the publish profile that you just downloaded.
6. On the Connection tab, update the Destination URL so that it is https, for example https://todolistservice-skwantoso.azurewebsites.net.  Click Next.
7. On the Settings tab, make sure Enable Organizational Authentication is NOT selected.  Click Publish.
8. Visual Studio will publish the project and automatically open a browser to the URL of the project.  If you see the default web page of the project, the publication was successful.

### Update the TodoListClient to call the TodoListService Running in Azure Web Sites

1. In Visual Studio, go to the TodoListClient project.
2. Open `app.config`.  Only one change is needed - update the `todo:TodoListBaseAddress` key value to be the address of the website you published, e.g. https://todolistservice-skwantoso.azurewebsites.net.
3. Run the client!  If you are trying multiple different client types (e.g. .Net, Windows Store, Android, iOS) you can have them all call this one published web API.

NOTE:  Remember, the To Do list is stored in memory in this TodoListService sample.  Azure Web Sites will spin down your web site if it is inactive, and your To Do list will get emptied.  Also, if you increase the instance count of the web site, requests will be distributed among the instances and the To Do will not be the same on each instance.

## About The Code

Coming soon.

## How To Recreate This Sample

coming soon.
