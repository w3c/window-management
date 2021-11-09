---
recipe: api-interface
title: 'Screens'
mdn_url: /en-US/docs/Web/API/Screens
specifications: https://webscreens.github.io/window-placement/#screens
browser_compatibility: api.Screens
---

## Description

The `Screens` interface of the Window Placement API provides multi-screen information and change events.

## Constructor

None.

## Properties

**`Screens.screens`**

Returns an array of `ScreenDetailed` objects, which describe each connected screen.

**`Screens.currentScreen`**

Returns a `ScreenDetailed` object for the current screen.

## Eventhandlers

 **`Screens.onscreenschange`**

Called when `screens` changes.

Note that this is not called on changes to the attributes of individual screens. Use `Screen.onchange` to observe those changes.

**`Screens.oncurrentscreenchange`**

Called when any attribute on `currentScreen` changes.

## Examples

### Logging the number of connected screens

The following example shows how to count the number of connected screens,
or log a warning if the permission is denied. The `getScreenDetails()` call
may show a permission prompt if necessary.

```js
if (screen.isExtended) {
  window.getScreenDetials().then(
    screenDetails => {
      console.log("Screens: " + screenDetails.screens.length);
    },
    error => {
      console.warn("Permission denied");
    }
  );
}
```

### Detecting when the set of screens change

The following example logs a message if the set of available screens
changes. This example assumes the permission was granted.

```js
window.getScreenDetails().then(
  screenDetails => {
    screenDetails.onscreenchange = event => {
      console.log("screens changed");
    };
  }
);
```

### Detecting when the current screen changes

The following example logs a message if the current screen changes,
such as when the window has moved to another screen.

```js
window.getScreenDetails().then(
  screenDetails => {
    let id = screenDetails.currentScreen.id;
    screenDetails.oncurrentscreenchange = event => {
      if (id !== screenDetails.currentScreen.id) {
        console.log("current screen changed");
        id = screenDetails.currentScreen.id;
      }
    };
  }
);
```
