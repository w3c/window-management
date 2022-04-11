## API Surface Changes

This file describes API surface changes made during experimentation. These
changes were made to address some feedback and requests:
- Github issue [30](https://github.com/w3c/window-placement/issues/30)
presented compelling API design advice from @jakearchibald and @kenchris:
  - Permission-gate access to EventTarget for screen array changes
  - Ensure data is updated (& sync accessible...) when change events are fired
  - Resolve unclear permission-gating of isMultiScreen()
- GitHub issues [35](https://github.com/w3c/window-placement/issues/35)
and [21](https://github.com/w3c/window-placement/issues/21) suggest
aligning related proposals, partly motivating a new Screens interface.
  - See the proposed [Screen Fold API](https://w3c.github.io/screen-fold/) and
  [Window Segments Enumeration API](https://github.com/webscreens/window-segments)
  - See the high-level [Form Factors Explainer](https://webscreens.github.io/form-factors/)
- Partner feedback requested access to:
  - User-friendly screen labels, which vastly improve custom multi-screen picker and configuration UIs for apps
  - Screen change events, including changes to the multi-screen bit, to obviate polling
- Chrome Security requested that this feature be disabled by default on embedded pages via a [permissions policy](https://w3c.github.io/webappsec-permissions-policy/)
  - In the second origin trial, cross origin iframes need to specify an `allow="window-placement"` attribute.
- Some screen properties of reduced imperative were removed, at least for now.
  - `id`: A temporary generated per-origin unique ID; reset when cookies are deleted.
  - `touchSupport`: Whether the screen supports touch input; a predecessor of `pointerTypes`.
  - `pointerTypes`: The set of PointerTypes supported by the screen.

## Examples of requesting additional screen information

**Before (1st Origin Trial, Chrome M86-M88)**

[This explainer snapshot](https://github.com/w3c/window-placement/blob/a1e6c7cbf6e60ca04483ef817c5ea0ff069beecd/EXPLAINER.md)
captures our older vision for the API, implemented in early experimentation.

```javascript
// Detect the presence of extended screen areas; async method on Window.
const multiScreenUI = await window.isMultiScreen();

// Request a static array of static dictionaries.
let cachedScreens = await window.getScreens();
cachedScreens[0].primary;  // e.g. true
cachedScreens[0].internal;  // e.g. true
cachedScreens[0].touchSupport;  // e.g. true
cachedScreens[0].scaleFactor;  // e.g. 2

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

**After (2nd Origin Trial, Chrome M93-M96)**

The [explainer](https://github.com/w3c/window-placement/blob/master/EXPLAINER.md)
and [draft spec](https://webscreens.github.io/window-placement/) capture our
latest vision for the API, implemented in later experimentation.

```javascript
// Detect the presence of extended screen areas; sync attribute on Screen.
const multiScreenUI = window.screen.isExtended;

// Request an interface with a FrozenArray of live Screen-inheriting objects.
const screenDetails = await window.getScreenDetails();
screenDetails.screens[0].isPrimary;  // e.g. true
screenDetails.screens[0].isInternal;  // e.g. true
screenDetails.screens[0].devicePixelRatio;  // e.g. 2
// New access to user-friendly labels for each screen:
screenDetails.screens[0].label;  // e.g. 'Samsung Electric Company 28"'

// Access the live object corresponding to the current `window.screen`.
// The object is updated on cross-screen window placements or device changes.
screenDetails.currentScreen;

// Observe 'screens' array and 'currentScreen' changes; sync info access.
let cachedScreenCount = screenDetails.screens.length;
let cachedCurrentScreen = screenDetails.currentScreen;
screenDetails.addEventListener('screenschange', function() {
  // NOTE: Does not fire on changes to attributes of individual Screens.
  let screenCountChanged = cachedScreenCount != screenDetails.screens.length;
});
screenDetails.addEventListener('currentscreenchange', function() {
  // NOTE: Fires on currentScreen attribute changes, and when the window moves to another screen.
  let currentScreenChanged = cachedCurrentScreen !== screenDetails.currentScreen;
});

// Observe changes to a specific Screen's attributes.
window.screen.addEventListener('change', function() { ... });
screenDetails.screens[0].addEventListener('change', function() { ... });
```
