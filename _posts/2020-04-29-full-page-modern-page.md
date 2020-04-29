---
layout: post
title:  "Displaying a modern page without header and navigation"
author: "Rune Sperre"
date:   2020-04-29 07:00:00 +0200
tags: [SharePoint, CSS]
---

### Do you miss the ability to use `?IsDlg=1` on your modern pages?

For those not familiar with that URL parameter, it is what SharePoint use internally when displaying things like forms in dialogs when in the classic view. It will display the form in a dialog without the header and navigation. The ribbon will still be there though.

By embedding our modern page in a classic page, we can make this happen by adding ?IsDlg=1 to the URL of the *classic* page.

#### The result:

![IsDlg on a modern page](/images/2020-04-29/2020-04-29-09-12-57.png)

A question was asked on [SharePoint StackExchange](https://sharepoint.stackexchange.com/) that brought me to this quite dirty solution. 

I had noticed that when embedding a SharePoint page in Teams the navigation and header was automatically stripped away for us, so I tried to figure out what caused this to happen.

There was nothing added to the URL that could help us, so it had to be the page being in an iframe that did the trick.

### The solution

I created a classic web part page and added a Script Web Part to it. 

If you don't have the Pages library available you can activate the two SharePoint Publishing Infrastructure features under Site Settings and it should be created for you.

In the script web part, i added the following code:

```javascript
<iframe id="embedframe" style="overflow:hidden;height:100%;width:100%" height="100%" width="100%" frameborder="0" src="https://contoso.sharepoint.com/sites/Playground/SitePages/Embed-this.aspx"></iframe>
<script src="//ajax.googleapis.com/ajax/libs/jquery/1.11.1/jquery.min.js"></script>

<script type="text/javascript">
_spBodyOnLoadFunctionNames.push("reallyhide");

function reallyhide(){
 if(window.location.search.toLowerCase().indexOf("isdlg=1") > 0){
   $("#s4-ribbonrow").hide();
   $("#embedframe").css({"position":"absolute","top":"0","left":"0"});
 }
}
</script>
<style>
#s4-bodyContainer {height:2000px;}
</style>
```
In the `iframe src attribute` I am pointing at a modern page that I want to display full screen. 

The javascript and jQuery bits will hide the ribbon and fix positioning issues once the `IsDlg` parameter is in place. I also had to force the bodyContainer to have a fixed height as well.

On classic pages, the css classes and element IDs won't change. Too many solutions have been buildt through the years that are dependent on these staying the same.

---

So there you have it. If you want to display a modern page without header and navigation, but don't want to touch the dynamic css classes of the modern world, this approach *should* work for quite some time.



