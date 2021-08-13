---
recipe: api-interface
title: 'ScreenAdvanced'
mdn_url: /en-US/docs/Web/API/ScreenAdvanced
specifications: https://webscreens.github.io/window-placement/#screenadvanced
browser_compatibility: api.ScreenAdvanced
---

## Description

The `ScreenAdvanced` interface of the Window Placement API extends the
`Screen` interface and provides additional per-screen information.

The `ScreenAdvanced` interface is a child of `Screen` and inherits its members.

## Constructor

None.

## Properties

_Inherits properties from its parent, `Screen`._

**`ScreenAdvanced.left`**

Returns the distance in pixels to the left edge of the screen from the primary screen origin.

**`ScreenAdvanced.top`**

Returns the distance in pixels to the top edge of the screen from the primary screen origin.

**`ScreenAdvanced.isPrimary`**

Returns true if this screen is the primary screen.

**`ScreenAdvanced.isInternal`**

Returns true if this screen is built into the device, like a laptop display.

**`ScreenAdvanced.devicePixelRatio`**

Returns the ratio of this screen's resolution in physical pixels to its resolution in CSS pixels.

**`ScreenAdvanced.id`**

Returns a temporary, generated per-origin unique ID; resets when cookies are deleted.

**`ScreenAdvanced.label`**

Returns a user-friendly label string for the screen, determined by the user agent and OS.

**`ScreenAdvanced.pointerTypes`**

Returns an array of `PointerType` strings representing input modes supported by the display.

## Examples

### Detailed information about the current screen

The following example shows how to query the browser for more
information about the current screen. The example assumes that
permission is granted.

```js

window.getScreens().then(
  screens => {
    var screen = screens.currentScreen;
    console.log("Label: " + screen.label);
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

### Detailed information about the available screens

The following example shows how to query the browser for more
information about all available screens. The example assumes that
permission is granted.

```js

window.getScreens().then(
  screens => {
    var availableScreens = screens.screens;
    availableScreens.forEach(screen => {
      console.log("Label: " + screen.label);
      console.log("  ID: " + screen.id);
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
