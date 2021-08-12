---
recipe: api-interface
title: 'Screens.screens'
mdn_url: /en-US/docs/Web/API/Screens/screens
specifications: https://webscreens.github.io/window-placement/#dom-screens-screens
browser_compatibility: api.Screens.screens
---

## Description

The `screens` read-only property of the `Screens` interface returns an array of `ScreenAdvanced` objects which describe the available screens.

## Syntax

`var _availableScreens_ = Screens.screens`

### Value

An frozen array of `ScreenAdvanced` objects.

## Examples

### Detailed information about the available screens

The following example shows how to query the browser for more
information about all available screens. The example assumes
that permission is granted.

```js

window.getScreens().then(
  screens => {
    var availableScreens = screens.screens;

    availableScreens.forEach(screen => {
      console.log("ID: " + screen.id);
      console.log("  Size: " + screen.width + " x " + screen.height);
      console.log("  Position: " + screen.left + " x " + screen.top);
      console.log("  Scale: " + screen.devicePixelRatio);
      console.log("  Primary? " + screen.isPrimary);
      console.log("  Internal? " + screen.isInternal);
      console.log("  Touch? " + screen.pointerTypes.includes("touch"));
    });
  }
);
```
