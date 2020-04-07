---
layout: post
title:  "Open a specific Excel Worksheet in the browser"
author: "Rune Sperre"
date:   2020-04-07 20:00:00 +0200
tags: [Excel, SharePoint]
---
## It is possible to have a link direcly to a sheet in an Excel Workbook in SharePoint.

There are three main ways of linking to a document in SharePoint:

1.  By using the **Copy Link** control
2.  By copying the URL when viewing the spreadsheet in the browser
3.  By using a hard link to the document using its path

The following method only works for the two first types of links, that looks like this:

````
(1)
https://yourtenant.sharepoint.com/:x:/s/sitename/EaPm6BD3KkVJvwjuzQdOrcEBR2YOqbFUWn22YGyL26cMKQ?e=8y1CEl

(2)
https://yourtenant.sharepoint.com/:x:/r/sites/sitename/_layouts/15/Doc.aspx?sourcedoc=%7B10E8E6A3-2AF7-4945-BF08-EECD07AAADC1%7D&file=Test.xlsx&action=default&mobileredirect=true&cid=2652a1f5-62b5-4d13-8caf-c1771a74f2cb
````

For both of these, if we add **&ActiveCell=Sheetname** it will  open the sheet we specify. We can also specify a cell in that sheet if we want to. Example:
   
````
https://yourtenant.sharepoint.com/:x:/s/sitename/EaPm6BD3KkVJvwjuzQdOrcEBR2YOqbFUWn22YGyL26cMKQ?e=8y1CEl&ActiveCell=Sheet2!C3
````
![ActiveCell](/images/20200407-excel.png)

Pretty neat!