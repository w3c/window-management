---
recipe: api-interface
title: 'Screen.isExtended'
mdn_url: /en-US/docs/Web/API/Screen/isExtended
specifications: https://webscreens.github.io/window-placement/#dom-screen-isextended
browser_compatibility: api.Screen.isExtended
---

## Description

The `isExtended` read-only property of the `Screen` interface returns true if the device has multiple screens that can be used for window placement.

## Syntax

`var _extended_ = screen.isExtended`

### Value

A boolean.

## Examples

The following example shows how to only query for screens if the
system has multiple screens that can be used for window placement, to
avoid showing a permission prompt if the system only has one.

```js
if (screen.isExtended) {
  console.log("Multiple screens available");
  window.getScreens().then(screens => {
    // ...
  });
}

```
