---
layout: post
title:  "Azure App Authentication in an Azure Function"
author: "Rune Sperre"
date:   2020-04-17 12:00:00 +0200
tags: [PnP, PowerShell, Azure]
---

This will briefly explain how to handle authentication in an Azure Function that uses the PnP PowerShell module. 

For information on how to generate an Azure AD App and certificate, check my previous post:  
[Azure AD App authentication in PnP PowerShell](/2020/04/17/azure-ad-app-authentication.html)

## Key Vault

To keep our certificate safe, we will upload it into an [Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/general/basic-concepts).

Press **Generate/Import** and upload your certificate:

![](/images/2020-04-17/2020-04-17-11-43-09.png)

Verify that it was successfully imported:
![](/images/2020-04-17/2020-04-17-11-42-26.png)

## Function App

Now we can import this certificate to our Function App:

![](/images/2020-04-17/2020-04-17-azure-function.png)

Under *Platform features*, select *SSL*. Here, go to *Private Key Certificates (.pfx)* and use the *Import Key Vault Certificate* button to select the previously imported certificate.

### Using the certificate in the function app

First we must load the certificate into our app. We do this by adding the application setting `WEBSITE_LOAD_CERTIFICATES` that points to the thumbprint of our certificate:
![](/images/2020-04-17/2020-04-17-12-53-20.png)

To get the PnP PowerShell module working we have to upload it into a folder called `modules` in our function.

**Note**: PnP PowerShell only works with runtime version ~1, be sure to set it to that before creating your function. It will **not** be possible to downgrade it if you have created a function in ~2 or ~3.

![](/images/2020-04-17/2020-04-17-12-39-12.png)


The code I am running is really simple:
```powershell
Connect-PnPOnline -Url https://mytenant.sharepoint.com -Tenant mytenant.onmicrosoft.com -ClientId 2cd1a65a-dc9d-4375-813b-fdc5gf20a6b8 -Thumbprint C3CA6F2F7B33CB4928FDCCC31206CA26FEB52648
Add-PnPListItem -List URL -Values @{"Title" = "Test from Azure Function"}
Disconnect-PnPOnline
```
*Client Id and thumbprints have obviously been faked here*  

Running it shows great success:

![](/images/2020-04-17/2020-04-17-12-56-43.png)
