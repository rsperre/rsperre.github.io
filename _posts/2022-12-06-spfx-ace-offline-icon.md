---
layout: post
title:  "Offline icon in Teams app for custom Adaptive Card"
author: "Rune Sperre"
date:   2022-12-06 07:00:00 +0200
tags: [SPFx, ACEs, Adaptive Cards]
---

When creating a custom Adaptive Card using SPFx, we have the choice of specifying the `officeFabricIconFontName` for the solution, which will used as the icon displayed in the top left corner of the card (and in the picker) when deployed.

Here is one using the default SharePointLogo icon:

![](/images/sharepointlogoicon.png)  

However, this icon will show as offline when displaying the card in the Teams app on Android and iOS: 

![](/images/offlineicon.png)  


According to a comment on this [GitHub issue](https://github.com/SharePoint/sp-dev-docs/issues/7772#issuecomment-1068272650), for now only the icons displayed in the custom card designer are supported. Pick one of those and you should be fine.

![](/images/carddesignericons.png)  
Still a few to choose from!