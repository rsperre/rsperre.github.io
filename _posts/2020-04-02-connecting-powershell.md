---
layout: post
title:  "Supplying credentials for Connect-PnPOnline"
author: "Rune Sperre"
date:   2020-04-02 12:00:00 +0200
tags: [PnP, PowerShell, SharePoint]
---

# Authentication

When connecting with `Connect-PnPOnline` we have some options on how to supply our credentials. As I am using this command quite often it is nice to be able to save a few seconds each time. These are the ones I find myself using most of the time.


## Azure AD App Authentication
In the april 2020 update of PnP PowerShell a new command was included to make the process of setting up an Azure AD App with certificate for authentication a whole lot easier. With the certificate in place, we connect like this:

```powershell
Connect-PnPOnline -Url https://mytenant.sharepoint.com -Tenant mytenant.onmicrosoft.com -ClientId 2cd1ff5a-dc9d-4ff5-813b-fdc5effff6b8 -Thumbprint C3CA6F5F7B33CB4908FDCFFEE9060A26FEB52648
```

See also my post: [Azure AD App authentication in PnP PowerShell](/2020/04/17/azure-ad-app-authentication.html).

## Password prompt
Well. That's basically the same you get if you do not supply anything for `-Credentials` at all...  
I am including it here because it can be useful in a script that different people in a team will be using, and need to use their own credentials.
```powershell
$credentials = Get-Credential
Connect-PnPOnline -Credentials $credentials -Url https://tenant.sharepoint.com/sites/demo
```

## Managed Credentials (Windows)
This is my go-to method for authenticating on the fly. In Credential Manager I have defined a number of generic credentials that I can use without getting a login window. This makes it easy to swich between my dev tenant and different live environments. Just give the credential a name that is easy to remember:

![Credential Manager](/images/20200402-auth01.png)

Then this name can be used directly in your connection:
```powershell
Connect-PnPOnline -Credentials MyDevEnvironment -Url https://mydevenvironment.sharepoint.com/sites/demo
```
## SharePoint AppId / AppSecret
Useful in its own way, it has the benefit of leaving your (admin) account name away from modified information and such, so noone knows who to blame! This can also be useful in an Azure Function, just be sure to protect the Id/Secret using the key vault or similar.

![SharePoint AppId](/images/20200402-auth02.png)

Note! This will limit what you do to the permission scope you have given the app registration.  
Anything outside of SharePoint will **NOT** work, for example creating an Office 365 Group connected Team Site as that requires access to Azure AD.

```powershell
$appid = "293e3320-2776-45e6-98e3-ff6c27a95b2f"
$appsecret = "oghiSf2QbTOL1ei3ZjQmHjogOjhuRnoNK2zeivcScO0="
$url = "https://mydevenvironment.sharepoint.com/sites/demo"

Connect-PnPOnline -ClientId $appid -ClientSecret $appsecret -Url $url
```

`Connect-PnPOnline` has quite a few other options for authenticating as well, I have covered the ones I find most useful for day-to-day operations.  
The full documentation of options be found here: [https://docs.microsoft.com/en-us/powershell/module/sharepoint-pnp/connect-pnponline](https://docs.microsoft.com/en-us/powershell/module/sharepoint-pnp/connect-pnponline)