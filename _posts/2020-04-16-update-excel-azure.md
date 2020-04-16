---
layout: post
title:  "Scheduled update of an Excel file from Azure Function"
author: "Rune Sperre"
date:   2020-04-16 18:00:00 +0200
tags: [Excel, SharePoint, Azure, PowerShell, PnP]
---

A while back I was asked to help with the following scenario: An Excel sheet needed to be updated and saved daily with fresh data from a datasource that required login. 

This was done manually.

Not only that, but after the data had been refreshed the data source and query had to be removed from the Excel file, to prevent users trying to update the data as this should be a snapshot of the data from a specific point in time.

The file was to be stored in a SharePoint library so that it could be easily accessed by all users.

---

## The Solution

I came across a really interesting PowerShell Module that helped me solve this one: [ImportExcel](https://github.com/dfinke/ImportExcel).  
This excellent module, developed by [Douglas Finke](https://github.com/dfinke/ImportExcel), lets you read and write Excel files without having to have a copy of Excel installed, which is perfect for an Azure Function!

I needed to create a PowerShell script that did the following three things:  
 - Read data from the SQL Server
 - Push data to an Excel sheet
 - Upload the Excel file to SharePoint

### Prerequisites

Before diving into the good bits we need to have some things in place.

#### Register a SharePoint App for writing the generated Excel file to a document library

We register apps by using `/_layouts/AppRegNew.aspx` on our target site :

![](/images/2020-04-16/2020-04-16-21-38-30.png)

Be sure to copy the Client Id and Client Secret!

Using the Client Id from the previous step, we grant the appropriate permissions to our app, using `/_layouts/AppInv.aspx`. We just need write access to the list, so we specify that with the following permission request XML:

```xml
<AppPermissionRequests AllowAppOnlyPolicy="true">  
   <AppPermissionRequest Scope="http://sharepoint/content/sitecollection/web/list" 
    Right="Write" />
</AppPermissionRequests>
```

Paste the Client Id from step 1 into the App Id field and hit "Lookup", then paste the permission request XML:

![](/images/2020-04-16/2020-04-16-21-48-57.png)

Since we specified **list** permission in our request, SharePoint asks which list the access should be granted for:

![](/images/2020-04-16/2020-04-16-21-45-44.png)

#### Prepare our Azure Function 

Since we are using two modules here, PnP PowerShell and ImportExcel, we need to upload those to our function. Any modules found in our functions `modules` folder will be loaded automatically:

![](/images/2020-04-16/2020-04-16-22-05-22.png)

I've found the easiest way to get these is by using the `Export-Module` command in PowerShell, for example: 

```powershell
Save-Module -Name SharePointPnPPowerShellOnline -Path c:\temp
```
This will save the files we need in a folder (the module name) under our specified path. 
Using the debug console in Kudu lets you drag & drop the files easily to the function. The path will be `D:\home\site\wwwroot\FunctionName`.

Next, we need to add the environment variables needed for the script. In this example we have the AppId and AppSecret for the SharePoint app, as well as a SQL User and password. 
In this example they are called "SPO_AppId", "SPO_AppSecret", "SQL_User" and "SQL_Pass".

#### The script

This is a stripped down version of the final result to keep it simple. In my real environment I am also doing some pre-processing of the data returned from the SQL Query.
```powershell
$outpath = "D:\local\temp"
$templatepath = "D:\home\site\wwwroot\DEMO_Excel_Update\template"
$siteurl = "https://contoso.sharepoint.com/sites/targetsite"
$connectionString = "User Id=$env:SQL_User;Password=$env:SQL_Pass;Server=tcp:demo.database.windows.net,1433;Initial Catalog=test_db;Persist Security Info=False;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;"
Write-Output "PowerShell Timer trigger function executed at:$(get-date)";

function processResults() {
    param ([object] $datarows)

    try {
        # Create a copy of the template file in the output folder
        Copy-Item -Path "$templatepath\Template.xlsx" -Destination "$outpath\Output.xlsx" -Force
    
        Write-Output "Exporting data to Excel..."
        $datarows | Export-Excel -Path "$outpath\Output.xlsx" -WorksheetName "DemoData"  -TableName "DemoTable" -ClearSheet -HideSheet "DemoData"
        
        Write-Output "Copying file to SharePoint..."
        Add-PNPFile -Path "$outpath\Output.xlsx" -Folder "Documents"
    }
    catch {
        $error[0]|format-list -force
    }
}

function executeQuery() {
    Write-Output "Getting data from Azure SQL..."
    try {
        $SqlConnection = New-Object System.Data.SqlClient.SqlConnection
        $SqlConnection.ConnectionString = $connectionString
        $SqlCmd = New-Object System.Data.SqlClient.SqlCommand
        $SqlCmd.CommandText = "select top 1000 * from DemoTable"
        $SqlCmd.Connection = $SqlConnection
        $SqlAdapter = New-Object System.Data.SqlClient.SqlDataAdapter
        $SqlAdapter.SelectCommand = $SqlCmd
        $DataSet = New-Object System.Data.DataSet
        $SqlAdapter.Fill($DataSet)
        processResults $DataSet.Tables[0].Rows 
        $SqlConnection.Close()
    }
    catch {
        Write-Output "Error getting data from Azure SQL:"
        Write-Output $_
    }
}

Connect-PNPOnline -AppId $env:SPO_AppId -AppSecret $env:SPO_AppSecret -Url $siteurl
executeQuery
Disconnect-PnPOnline
```
---
### A closer look at The `Export-Excel` command

    $datarows | Export-Excel -Path "$outpath\Output.xlsx" -WorksheetName "DemoData"  -TableName "DemoTable" -ClearSheet -HideSheet "DemoData"

This will take the `$datarows` object and pipe it into the `Export-Excel`, using the file we copied ($outpath\Output.xlsx) as the source. It will insert a new worksheet called "DemoData" and give the inserted table a name (DemoTable). Then it will hide the sheet.  

The reason for this is that the template file that was copied will do some furher magic using the inserted data and give it proper formatting and such. The Export-Excel command will only touch the specified worksheet.

---

### Testing it locally
Simply add the same keys with the same names to your local environment and the script will run just fine locally as well! 