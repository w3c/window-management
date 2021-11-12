---
recipe: api-interface
title: 'ScreenDetails.currentScreen'
mdn_url: /en-US/docs/Web/API/ScreenDetails/currentScreen
specifications: https://webscreens.github.io/window-placement/#dom-screendetails-currentscreen
browser_compatibility: api.ScreenDetails.currentScreen
---

## Description

The `currentScreen` read-only property of the `ScreenDetails` interface returns
a `ScreenDetailed` object giving more details about the current screen
than is available on the `window.screen` object.

## Syntax

`var _screen_ = ScreenDetails.currentScreen`

### Value

A `ScreenDetailed` object.

## Examples

### Detailed information about the current screen

The following example shows how to query the browser for more
information about the current screen. The example assumes that
permission is granted.

```js
window.getScreenDetails().then(
  screenDetails => {
    var screen = screenDetails.currentScreen;

    console.log("Label: " + screen.label);
    console.log("Size: " + screen.width + " x " + screen.height);
    console.log("Position: " + screen.left + " x " + screen.top);
    console.log("Scale: " + screen.devicePixelRatio);
    console.log("Primary? " + screen.isPrimary);
    console.log("Internal? " + screen.isInternal);
  }
);
```
