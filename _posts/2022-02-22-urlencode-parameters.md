---
layout: post
title:  "Get-PnPMicrosoft365Group - Invalid filter clause"
author: "Rune Sperre"
date:   2022-02-22 22:22:22 +0200
tags: [PnP, PowerShell]
---

Are you getting errors trying to retrieve a Microsoft 365 group using PnP PowerShell? The identity you are using might contain characters that will break the filter string being passed to Microsoft Graph (in this example, the ampersand (&)):

```error
❯ Get-PnPMicrosoft365Group -Identity "People & Places"
Get-PnPMicrosoft365Group : Invalid filter clause
At line:1 char:1
+ Get-PnPMicrosoft365Group -Identity "People & Places"
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) [Get-PnPMicrosoft365Group], PSInvalidOperationException
    + FullyQualifiedErrorId : InvalidOperation,PnP.PowerShell.Commands.Microsoft365Groups.GetMicrosoft365Group
``` 

**Solution**
The name needs to be encoded - this can be done using System.Web.HttpUtility:
```powershell
$name = "People & Places"
$group = Get-PnPMicrosoft365Group -Identity ([System.Web.HttpUtility]::UrlEncode($name))
```
This will turn "People & Places" into "People+%26+Places".
