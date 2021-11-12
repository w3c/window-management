---
recipe: api-interface
title: 'ScreenDetailed'
mdn_url: /en-US/docs/Web/API/ScreenDetailed
specifications: https://webscreens.github.io/window-placement/#screendetailed
browser_compatibility: api.ScreenDetailed
---

## Description

The `ScreenDetailed` interface of the Window Placement API extends the
`Screen` interface and provides additional per-screen information.

The `ScreenDetailed` interface is a child of `Screen` and inherits its members.

## Constructor

None.

## Properties

_Inherits properties from its parent, `Screen`._

**`ScreenDetailed.left`**

Returns the distance in pixels to the left edge of the screen from the primary screen origin.

**`ScreenDetailed.top`**

Returns the distance in pixels to the top edge of the screen from the primary screen origin.

**`ScreenDetailed.isPrimary`**

Returns true if this screen is the primary screen.

**`ScreenDetailed.isInternal`**

Returns true if this screen is built into the device, like a laptop display.

**`ScreenDetailed.devicePixelRatio`**

Returns the ratio of this screen's resolution in physical pixels to its resolution in CSS pixels.

**`ScreenDetailed.label`**

Returns a user-friendly label string for the screen, determined by the user agent and OS.

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

### Detailed information about the available screens

The following example shows how to query the browser for more
information about all available screens. The example assumes that
permission is granted.

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
