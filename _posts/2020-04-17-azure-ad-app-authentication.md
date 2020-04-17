---
layout: post
title:  "Azure AD App authentication in PnP PowerShell"
author: "Rune Sperre"
date:   2020-04-17 06:00:00 +0200
tags: [PnP, PowerShell, Azure]
---

In the [April 2020 Release](https://github.com/pnp/PnP-PowerShell/releases/tag/3.20.2004.0) of PnP PowerShell a new command was introduced:

 > Initialize-PnPPowerShellAuthentication

This command lets you set up a new Azure AD App with a self-signed certificate in just a few minutes.

Here's how that looks:
```powershell
> Initialize-PnPPowerShellAuthentication -ApplicationName "MyAdminApp" -Tenant "mytenant.onmicrosoft.com"
Waiting 60 seconds to launch consent flow in a browser window. This wait is required to make sure that Azure AD is able to initialize all required artifacts.........................


AzureAppId                           Certificate Thumbprint
----------                           ----------------------
cefa6dff-196b-4497-99de-ff79e3b2b385 FF82F0195516E0FFF658FBE1F8081C6029DC80A9
```
This will create an app registration in Azure AD, attach a self signed certificate to it, then request the needed permissions. We can see that the app is in place before granting the permissions using the permission request pop-up:

![](/images/2020-04-17/2020-04-17-07-38-36.png)

In this process I got three pop-ups:

 1. Initial browser authentication
 2. Selection of account for accepting permission request
 3. The permission request itself:

![](/images/2020-04-17/2020-04-17-07-39-13.png)

**...and that's it! Now we can use this for authentication:**

```powershell
❯ Connect-PnPOnline -Url https://mytenant.sharepoint.com -Tenant mytenant.onmicrosoft.com -ClientId cefa6dff-196b-4497-99de-ff79e3b2b385 -Thumbprint FF82F0195516E0FFF658FBE1F8081C6029DC80A9
Connect-PnPOnline : Value cannot be null.
Parameter name: certificate
At line:1 char:1
+ Connect-PnPOnline -Url https://mytenant.sharepoint.com -Tenant mytenant ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Connect-PnPOnline], ArgumentNullException
    + FullyQualifiedErrorId : System.ArgumentNullException,SharePointPnP.PowerShell.Commands.Base.ConnectOnline
```
**...or maybe not...**

It seems the command failed to install the certificate on my machine. To get around this, I ran the command again and this time I included the parameter `-OutPath` :

    ❯ Initialize-PnPPowerShellAuthentication -ApplicationName "MyAdminApp4" -Tenant "mytenant.onmicrosoft.com" -OutPath c:\temp

I was expecting to have to install the certificate manually, but first I tried to connect again - and this time it worked!  

Notice that the name now ends with "4". I tried two other times without success first using other parameters like specifying which store the certificate should be added to and running in an elevated shell. 

Somehow outputting the certificate seemed to do the trick!

```powershell
❯ Connect-PnPOnline -Url https://mytenant.sharepoint.com -Tenant mytenant.onmicrosoft.com -ClientId 2cd1ff5a-dc9d-4ff5-813b-fdc5effff6b8 -Thumbprint C3CA6F5F7B33CB4908FDCFFEE9060A26FEB52648
❯ Add-PnPListItem -List Test -Values @{"Title" = "Test from AAD App"}

Id    Title                                              GUID
--    -----                                              ----
3     Test from AAD App                                  e740a92c-3d7e-48cb-a841-5cb6a4471cda
```
And this is how it looks in SharePoint:

![](/images/2020-04-17/2020-04-17-08-27-53.png)

---

### Final thoughts

This makes the process of setting up an Azure AD App for authentication a whole lot easier. You don't have to generate a certificate or remember which permissions to grant the app.

For PowerShell scripts and Azure Functions this is the best way to handle authentication. You do not have to worry about MFA prompts, and even better - any visible changes (like in my list item example) will be made by "SharePoint App" instead of your (admin) account.

You can add multiple certificates to the same App, that can be useful if you are a team of admins / developers and you want to be able to revoke access.

Just remember to set a reminder for 10 years when the certificate expires!

---

Here is another post I made on other ways of connecting using `Connect-PnPOnline`:  
[Supplying credentials for Connect-PnPOnline](/2020/04/02/connecting-powershell.html)

