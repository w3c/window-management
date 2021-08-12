---
recipe: api-interface
title: 'Screens'
mdn_url: /en-US/docs/Web/API/Screens
specifications: https://webscreens.github.io/window-placement
browser_compatibility: api.Screens
---

## Description


The `Screens` interface of the Window Placement API provides multi-screen information and change events.

## Constructor

None.

## Properties

**`Screens.screens`**

Returns an array of `ScreenAdvanced` objects, which describe each connected screen.

**`Screens.currentScreen`**

Returns a `ScreenAdvanced` object for the current screen.

## Eventhandlers

**`Screens.onchange`**

Called when `screens` or `currentScreen` changes.

Note that this is not called on changes to the attributes of individual screens. `Screen.onchange` can be used to observe those changes.

## Examples

### Logging the number of connected screens

The following example shows how to count the number of connected screens,
or log a warning if the permission is denied. The `getScreens()` call
may show a permission prompt if necessary.

```js
if (screen.isExtended) {
  window.getScreens().then(
    screens => {
      console.log("Screens: " + screens.screens.length);
    },
    error => {
      console.warn("Permission denied");
    }
  );
}
```

### Detecting when screen properties change

The following example logs a message if the set of available screens
changes. This example assumes the permission was granted.

```js
window.getScreens().then(
  screens => {
    screens.onchange = event => {
      console.log("screens changed");
    };
  }
);
```

### Detecting when the current screen changes

The following example logs a message if the current screen changes,
such as when the window has moved to another screen.

```js
window.getScreens().then(
  screens => {
    let id = screens.currentScreen.id;
    screens.onchange = event => {
      if (id !== screens.currentScreen.id) {
        console.log("current screen changed");
        id = screens.currentScreen.id;
      }
    };
  }
);
```
