## API Surface Changes

This file enumerates changes made to the API surface.

## Requesting additional screen information

**Before (1st Origin Trial, Chrome M86-89)**
```javascript
// Detect the presence of extended screen areas; async method on Window.
const multiScreenUI = await window.isMultiScreen();

// Request a static array of static dictionaries.
let cachedScreens = await window.getScreens();
cachedScreens[0].primary;  // e.g. true
cachedScreens[0].internal;  // e.g. true
cachedScreens[0].touchSupport;  // e.g. true

// Access the static dictionary corresponding to the current `window.screen`.
// The object is stale on cross-screen window placements or device changes.
let currentScreen = cachedScreens.find(
    s => s.availLeft == window.screen.availLeft &&
    s.availTop == window.screen.availTop);

// Observe changes of one or more screens; need to await updated info.
window.addEventListener('screenschange', async function() {
  let updatedScreens = await window.getScreens();
  let screenCountChanged = screens.length != updatedScreens.length;
});
```

**After (2nd Origin trial, Chrome M9X)**
```javascript
// Detect the presence of extended screen areas; sync attribute on Screen.
const multiScreenUI = window.screen.isExtended;

// Request an interface with a FrozenArray of live Screen-inheriting objects.
let screensInterface = await window.getScreens();
screensInterface.screens[0].isPrimary;  // e.g. true
screensInterface.screens[0].isInternal;  // e.g. true
screensInterface.screens[0].pointerTypes;  // e.g. ["touch"]

// New access to user-friendly labels for each screen:
screensInterface.screens[0].label;  // e.g. 'Samsung Electric Company 28"'

// Access the live object corresponding to the current `window.screen`.
// The object is updated on cross-screen window placements or device changes.
screensInterface.currentScreen;

// Observe 'screens' array or 'currentScreen' ref changes; sync info access.
let cachedScreenCount = screensInterface.screens.length;
screensInterface.addEventListener('change', function() {
  // NOTE: Does not fire on changes to attributes of individual Screens.
  let screenCountChanged = cachedScreenCount != screensInterface.screens.length;
});

// Observe changes to a specific Screen's attributes.
screensInterface.screens[0].addEventListener('change', function() { ... });
window.screen.addEventListener('change', function() { ... });
```