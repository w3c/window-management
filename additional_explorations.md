# Additional Explorations around Window Placement

This document explores potential Window Placement use cases and solutions beyond
the scope of the current proposal. These examples require API or implementer
work that is **not** actively purused by the current proposal, but may be
included in future iterations of the proposal, or in separate proposals. These
explorations have helped to shape the initial Window Placement API design.

## Possible future goals of the [current proposal](https://github.com/webscreens/window-placement/blob/master/explainer.md)

Future goals may provide additional capabilities and more ergonomic APIs for web
applications to manage windows. See explorations of these tentative goals below.
* Offer more ergonomic APIs
* Extend window state and display APIs
* Surface events on window bounds or state changes
* Support multiple fullscreen elements from a single document
* Extend user activation affordances
* Allow window placements that span two or more screens

## Offer more ergonomic APIs

`window.open()` is generally poorly regarded for its string-based approach to
specifying features, which have been deprecated over the years. Additionally,
`window.open()` and `window.moveTo()` are synchronous, which does not reflect
suit the asynchronous behavior of underlying user agents and OS window managers,
and their coordinates do not clearly support multi-screen environments. Further,
they lack proper feature detection for specific supported behaviors.

There are several possible options to address these deficiencies:
* Overload `window.open()` with a dictionary parameter, modernizing the options
  available to callers and offering explicit multi-monitor support:
  * window.open(url, { name: 'name', features: '...', screen: bestScreen});
  * window.open(url, name, {screen: bestScreen, innerWidth: w, innerHeight: h});
  * Dictionary parameter allows developers to perform feature detection
  * Unclear if we could return a promise for this overload to make it async
* Overload `window.moveTo()` to accept a screen argument, treating the specified
  `x` and `y` coordinates as local to the target screen:
  * window.moveTo(x, y, {screen: targetScreens});
* Add a new async API to open windows, maybe extend (and expose on Window)
  [`Clients.openWindow()`](https://www.w3.org/TR/service-workers-1/#dom-clients-openwindow)
  method via a `windowOptions` dictionary parameter with multi-screen support:
  * window.openWindow(url, {left: x, top: y, width: w, height: h, screen: s});
    * Coordinates specified relative to the target screen OR:
    * Nix `screen` and specify coordinates relative to the primary screen
  * Consider offering support for window features, eg. maximized/fullscreen?
  * Should `width` and `height` be inner* or outer* values, or accept either?
  * Support existing `window.open()` features, and/or new features?
  * Async allows permission requests; dictionary allows feature detection
* Add a new async API to get/set `Window` bounds:
  * window.setBounds({left: x, top: y, width: w, height: h, screen: s});
  * Parallels proposed dictionaries for window opening above
  * Async allows permission requests; dictionary allows feature detection

## Extend window state and display APIs

For reference, the space of windows states considered includes:
* `normal`: Normal 'restored' state (a framed window not in another state)
* `fullscreen`: Content fills the display without a frame
* `maximized`: Framed content occupies the available screen space
* `minimized`: Window is hidden and can be re-shown through OS controls
* `snapLeft|Right`: Framed content occupies half of available screen space

Some possible ways that window state information and controls might be exposed:
* Show a new window with a specific state
  * Restore deprecated 'fullscreen="yes"' window.open() feature (w/permission?)
    * Apparently deprecated for abuse, poor ergonomics, and lack of support
    * Making a user-granted permission a prerequisite for this feature may help
  * Extend window.open() feature support for 'maximized="yes"' (w/permission?)
    * This may suffer from similar drawbacks as 'fullscreen="yes"' without care
  * Extend new open window function dictionary parameters with a `state` member
* Query or change an existing window's state
  * Support [window.minimize()](https://developer.mozilla.org/en-US/docs/Web/API/Window/minimize)
    and add similar methods to get/set individual window states
  * Add new methods to support getting or setting the window state value
  * Support explicit z-ordering, such as an `"alwaysOnTop"` window state
    * This is sensitive, and may require additional permissions/controls
* Observe window state changes with a `onwindowstate` event (see goal below too)

Window [display](https://developer.mozilla.org/en-US/docs/Web/Manifest/display)
modes may warrant similar runtime access patterns:
* `fullscreen`: Window content fills the display without a frame
* `standalone`: A standalone application frame
* `minimal-ui`: Similar to `standalone` with a minimal set of UI elements
* `browser`: A conventional browser tab

There are also proposals/explorations for new display modes or modifiers:
* [Window Controls Overlay](https://github.com/MicrosoftEdge/MSEdgeExplainers/blob/master/TitleBarCustomization/explainer.md)
* [Display Mode Override](https://github.com/dmurph/display-mode/blob/master/explainer.md)
* [Tabbed Application Mode](https://github.com/w3c/manifest/issues/737)
* [More precise control over app window features](https://github.com/w3c/manifest/issues/856)

Here are some possible use cases for the extended window state and display APIs:
* Open a new fullscreen slideshow window on anoter screen, keep current window
* Minimize associated windows when the users minimizes a 'main' window; eg:
  * Doctor minimizes patient case window, app minimizes associated image windows
* Web application offers settings to show or hide minimal-ui native controls
* Video conferencing window wishes to be [always-on-top](https://github.com/webscreens/window-placement/issues/10)

There are open questions around the value and uses cases here:
* Need additional attestation of compelling use cases from developers
* Need to assess the risks and mitigations versus the utility offered
* Consider creation and management of dependent or 'child' window types

Here are some basic examples of use cases solved by these types of APIs:

```js
// Open a new fullscreen slideshow window, use the current window for notes.
// FUTURE: Request fullscreen and a specific screen when opening a window.
window.openWindow(slidesUrl, { state: 'fullscreen', screen: externalScreen });
// OR: window.open(url, name, `left=${s.left},top=${s.top},fullscreen=1`);
// NEW: Add `screen` parameter on `requestFullscreen()`.
document.getElementById(`notes`).requestFullscreen({ screen: internalScreen });
```

```js
// Open a new maximized window with an image preview on a separate display.
// FUTURE: Request maximized and a specific screen when opening a window.
window.openWindow(imageUrl, { state: 'maximized', screen: targetScreen });
```

TODO: Add additional use cases and examples of how they would be solved.

## Surface events on window bounds or state changes

Currently, `Window` fires an event when content is resized:
[onresize](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onresize).
There may be value in firing events when windows move or when state changes:
* Add `"onmove"`, `"onwindowdrag"`, and `"onwindowdrop"` Window events
* Add `"onwindowstate"` Window event for state changes

This allows sites to observe window placement and state changes, useful for
recognizing, persisting, and restoring user preferences for specific windows.
This would also be useful in scenarios where the relative placement of windows
might inform automated placement of accompanying windows (eg. a grid of windows
or 'child' window behavior).

## Support multiple fullscreen elements from a single document

As noted in the explainer, it may not be feasible or straightforward for
multiple elements in the same document to show as fullscreen windows on separate
screens simultaneously. This is partly due to the reuse of the underlying
platform window as a rendering surface for the fullscreen window.

Support for this behavior may solve some multi-screen use cases, for example:

```js
const slides = document.querySelector("#slides");
const notes = document.querySelector("#notes");
// NEW: Add `screen` parameter on `requestFullscreen()`.
slides.requestFullscreen({ screen: screens[1] });
// FUTURE: Additional implementer work would be needed to support showing
// multiple fullscreen windows of separate elements in the same document.
notes.requestFullscreen({ screen: screens[0] });
```

## Extend user activation affordances

Currently, the transient user activation of a button click is consumed by
opening a window. That complicates or blocks scenarios that might wish to
also  request fullscreen from the event handler for a single user click.
Relatedly, most browsers allow sites to open multiple popup windows with a
single click if they have user permission.

Allowing new behaviors would solve some valuable use cases, for example:
* Requesting fullscreen and opening a new window (in either order)
  ```js
  // Open notes on the internal screen and slides on the external screen.
  // NEW: `left` and `top` may be outside the window's current screen.
  window.open(`./notes`, ``, `left=${s1.availLeft},top=${s1.availTop}`);
  // NEW: `screen` on `fullscreenOptions` for `requestFullscreen()`.
  document.getElementById(`notes`).requestFullscreen({ screen: s2 });
  ```
* Opening a new window, and requesting fullscreen for that window
  ```js
  // Open slides and request that they show fullscreen on the external screen.
  // NEW: `left` and `top` may be outside the window's current screen.
  const slides = window.open(`./slides`, ``, ``);
  // NEW: `screen` on `fullscreenOptions` for `requestFullscreen()`.
  slides.document.body.requestFullscreen({ screen: externalScreen });
  ```

## Allow window placements that span two or more screens

With the introduction of new foldable devices, and with some affordances of
having a single content window in multi-screen environments, it may become more
common for windows to span multiple displays. This may be worth considering as
new multi-screen aware Window Placement APIs are developed.

## Other explorations

### Allow sites to enumerate windows of the same origin or those they've opened

This may be useful as web applications support URL handling or launch events.
* Add `client.Enumerate()` to list existing windows from a Service Worker

### Swap fullscreen windows between displays

The proposal suggests that subsequent requestFullscreen calls targetting a new
screen should move the existing fullscreen window to the new target screen. This
is useful for slide show presentations, if users wish to change the screens used
for slides, notes, and controls after initially entering fullscreen.

```js
// Step 1: Slideshow starts with multi-window and multi-screen fullscreen.
// NEW: `requestFullscreen()` supports an optional screen parameter.
let screens = await getScreens();
slidesWindow.document.body.requestFullscreen({ screen: screens[0] });
notesWindow.document.body.requestFullscreen({ screen: screens[1] });

// Step 2: User selects a slideshow control setting to swap the screens.
// NEW: `requestFullscreen()` supports subsequent calls with different screens.
slidesWindow.document.body.requestFullscreen({ screen: screens[1] });
notesWindow.document.body.requestFullscreen({ screen: screens[0] });
```

Other prospective techniques (like exiting fullscreen, moving the window across
displays, and re-requesting fullscreen) would fail due to the consumption of
transient user activation and/or suffer from flickering of intermediate states,
not to mention the poor developer ergonomics of this process.

### Creation and management of dependent or 'child' window types

Dependent or child windows are useful in certain native application contexts:
* Creative apps (image/audio/video) with floating palettes, previews, etc.
* Medical applications with separate windows for case reports and images

```js
// Synchronously move palettes when the main editor moves.
const palette1 = window.open("/palette/1", "palette1", "alwaysOnTop");
const palette2 = window.open("/palette/2", "palette2", "alwaysOnTop");
window.addEventListener("move", event => {
  palette1.moveBy(event.deltaX, event.deltaY);
  palette2.moveBy(event.deltaX, event.deltaY);
});
```

```js
// Open dependent/child windows that the OS/browser moves with a parent window.
const palette1 = window.open("/palette/1", "palette1", "dependent=true");
```

### Finance applications with multiple dashboards

Starting the app opens all the dashboards across multiple screens:
```js
// Service worker script
self.addEventListener("launch", event => {
  event.waitUntil(async () => {
    const screens = self.screens;
    const maxDashboardCount = 5;
    const usedScreenCount = Math.min(screens.length, maxDashboardCount);
    for (let screen = 1; screen < usedScreenCount; ++screen) {
      await clients.openWindow(`/dashboard/${screen}`, screens[screen]);
    }
  });
});
```

Starting the app restores all dashboards' positions from the prior session:
```js
self.addEventListener("launch", event => {
  event.waitUntil(async () => {
    // Initialize IndexedDB database.
    const db = idb.open("db", 1, upgradeDb => {
      if (!upgradeDb.objectStoreNames.contains("windowConfigs")) {
        upgradeDb.createObjectStore("windowConfigs", { keyPath: "name" });
      }
    });

    // Retrieve preferred dashboard configurations.
    const configs = db.transaction(["windowConfigs"])
                      .objectStore("windowConfigs")
                      .getAll();
    for (let config : configs) {
      // Open each dashboard, assuming the user's screen setup hasn't changed.
      const dashboard = await clients.openWindow(config.url, config.screen);
      dashboard.postMessage(config.options);
    }
  });
});

// Record the latest configuration relayed by a dashboard that was just closed.
self.addEventListener("message", event => {
  idb.open("db", 1)
      .transaction(["windowConfigs"], "readwrite")
      .objectStore("windowConfigs")
      .put(event.data);
});
```

```js
// Dashboard script
window.name = window.location;

// Configure dashboard according to preferences relayed by the Service Worker.
window.addEventListener("message", event => {
  window.moveTo(event.data.left, event.data.top);
  window.resizeTo(event.data.width, event.data.height);
});

// Send dashboard's configuration to the Service Worker just before closing.
window.addEventListener("beforeunload", event => {
  const windowConfig = {
    name: window.name,
    url: window.location,
    options: {
      left: window.screenLeft,
      top: window.screenTop,
      width: window.outerWidth,
      height: window.outerHeight,
    },
    screen: window.screen,
  };
  navigator.serviceWorker.controller.postMessage(windowConfig);
});
```

Snap dashboards into place when moved, according to a pre-defined configuration:
```js
// Reposition/resize window into the nearest left/right half of the screen when
// programatically moved or manually dragged then dropped.
window.addEventListener("windowDrop", event => {
  const windowCenter = window.screenLeft + window.outerWidth / 2;
  const screenCenter = window.screen.availLeft + window.screen.availWidth / 2;
  const newLeft = (windowCenter < screenCenter) ? 0 : screenCenter;
  window.moveTo(newLeft, 0);
  window.resizeTo(window.screen.width, window.screen.height);
});
```

### Small form-factor applications, e.g. calculator, mini music player
Launch the app with specific (or bounded) dimensions:
```js
// Service worker script
self.addEventListener("launch", event => {
  event.waitUntil(async () => {
    // At most 800px wide.
    const width = Math.min(window.screen.availWidth * 0.5, 800);
    // At least 200px tall.
    const height = Math.max(window.screen.availHeight * 0.3, 200);

    window.resizeTo(width, height);
  });
});
```
