---
layout: post
title:  "JSON column formatting: toUpperCase()"
author: "Rune Sperre"
date:   2020-04-03 08:00:00 +0200
tags: [JSON, CSS]
---
I was checking the [list of supported operators](https://docs.microsoft.com/en-us/sharepoint/dev/declarative-customization/column-formatting#operators) and noticed there was a `toLowerCase()` but no `toUpperCase()`?

I was starting to consider an approach to create one by looping each character when it hit me....

```json
{
  "elmType": "span",
    "txtContent": "@currentField",
    "style": {
        "text-transform": "uppercase"
    }
}
```

It's easy to forget that there might be really simple workarounds instead of making things complicated!