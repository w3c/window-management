---
recipe: api-interface
title: 'Screens.currentScreen'
mdn_url: /en-US/docs/Web/API/Screens/currentScreen
specifications: https://webscreens.github.io/window-placement/#dom-screens-currentscreen
browser_compatibility: api.Screens.currentScreen
---

## Description

The `currentScreen` read-only property of the `Screens` interface returns
a `ScreenAdvanced` object giving more details about the current screen
than is available on the `window.screen` object.

## Syntax

`var _screen_ = Screens.currentScreen`

### Value

A `ScreenAdvanced` object.

## Examples



### Detailed information about the current screen

The following example shows how to query the browser for more
information about the current screen. The example assumes that
permission is granted.

```js

window.getScreens().then(
  screens => {

    var screen = screens.currentScreen;

    console.log("ID: " + screen.id);
    console.log("Size: " + screen.width + " x " + screen.height);
    console.log("Position: " + screen.left + " x " + screen.top);
    console.log("Scale: " + screen.devicePixelRatio);
    console.log("Primary? " + screen.isPrimary);
    console.log("Internal? " + screen.isInternal);
    console.log("Touch? " + screen.pointerTypes.includes("touch"));
  }
);
```
