---
recipe: api-interface
title: 'ScreenDetails.screens'
mdn_url: /en-US/docs/Web/API/ScreenDetails/screens
specifications: https://webscreens.github.io/window-placement/#dom-screendetails-screens
browser_compatibility: api.ScreenDetails.screens
---

## Description

The `screens` read-only property of the `ScreenDetails` interface returns an array of `ScreenDetailed` objects that describe the available screens.

## Syntax

`var _availableScreens_ = ScreenDetails.screens`

### Value

A frozen array of `ScreenDetailed` objects.

## Examples

### Detailed information about the available screens

The following example shows how to query the browser for more
information about all available screens. The example assumes
that permission is granted.

```js
window.getScreenDetails().then(
  screenDetails => {
    var availableScreens = screenDetails.screens;
    availableScreens.forEach(screen => {
      console.log("Label: " + screen.label);
      console.log("  Size: " + screen.width + " x " + screen.height);
      console.log("  Position: " + screen.left + " x " + screen.top);
      console.log("  Scale: " + screen.devicePixelRatio);
      console.log("  Primary? " + screen.isPrimary);
      console.log("  Internal? " + screen.isInternal);
    });
  }
);
```
