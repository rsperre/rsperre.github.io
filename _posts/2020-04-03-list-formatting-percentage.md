---
layout: post
title:  "JSON column formatting for percentages"
author: "Rune Sperre"
date:   2020-04-03 07:00:00 +0200
tags: [JSON]
---


### Here's a short one. How do we achieve this look on a number field with percentages?

![JSON Column formatting](/images/20200403-json-percentage.png)

Heres the needed JSON to do just that:
```json
{
  "elmType": "div",
  "children": [
    {
      "elmType": "span",
      "style": {
        "width": "100%",
        "background-color": "#f44"
      },
      "children": [
        {
          "elmType": "span",
          "txtContent": {
            "operator": "+",
            "operands": [{ "operator": "*", "operands": ["@currentField",100]}," %"]
          },
          "style": {
            "background-color": { "operator": "?", "operands": [
                { "operator": "<=", "operands": [ "@currentField",0.33 ] },
                   "#eeaa00",
                {
                  "operator": "?",
                  "operands": [{"operator": "<=","operands": ["@currentField",0.66]},
                    "#ffff00",
                    "#00ff00"
                  ]
                }
              ]
            },
            "color": { "operator": "?", "operands": [
            { "operator": "<=", "operands": [ "@currentField",0.33 ] },
                   "white","black"
            ]},
            "width": {
              "operator": "+",
              "operands": [
                {
                  "operator": "*",
                  "operands": [
                    "@currentField",
                    100
                  ]
                },
                "%"
              ]
            },
            "text-align": "center",
            "white-space": "nowrap",
            "display": "block"
          }
        }
      ]
    }
  ]
}
```

### How this works

- First we add a `span` element with a red background 
color.
- We add another `span` as a child, with [operators](https://docs.microsoft.com/en-us/sharepoint/dev/declarative-customization/column-formatting#operators) to decide the background color and width of the element. 
- The same logic is used to set the foreground color to be black or white to increase readability.

To construct this kind of logic, we use the syntax:
   
```json
"property": {
  "operator": "?",
  "operands": [
    {
      "operator": "<",
      "operands": [
        0, 1
      ]
    },
    "true",
    "false"
  ]
}
````
This would translate to:
```typescript
var property = (0 < 1) ? "true" : "false";
```