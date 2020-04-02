---
layout: post
title:  "Strange problem with executeFlow in customRowAction"
author: "Rune Sperre"
date:   2020-04-01 07:00:00 +0200
tags: [Bugs, Flow, Power Automate, JSON]
---

Came across this strange problem. 
You can add buttons to your lists that will start a flow for the item using JSON column formatting. That is quite cool and useful, but there is a strange bug when you filter the list by searcing.

**Example**
```json
{
  "elmType": "div",
  "children": [
    {
      "elmType": "button",
      "txtContent": "Start Flow",
      "customRowAction": {
        "action": "executeFlow",
        "actionParams": "{\"id\": \"d2098ee2-4f9d-4d84-8557-a39fbc249231\", \"headerText\":\"Start flow\",\"runFlowButtonText\":\"Just testing\"}"
      }
    },
    {
      "elmType": "button",
      "txtContent": "Open Item",
      "customRowAction": {
        "action": "defaultClick"
      }
    }
  ]
}
```
This will add two buttons to a list. The first one will start the defined flow with the id specified in `actionParams`, the second will open the item using the default click action:

![Custom buttons!](/images/customaction-01.png)

This works just fine in the standard view and when sorting and filtering. However, if we filter the list by searching, the first button stops working. The second one still works as expected.

![Custom buttons!](/images/customaction-02.png)

I will investigate this further, but I do not think this is something that end users can fix themselves.