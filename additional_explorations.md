# Additional Explorations of Window Placement on the Web

This document explores alternatives, possible supplements, other considerations,
and examples that have helped shape the current proposal, which is described in
[EXPLAINER.md](https://github.com/webscreens/window-placement/blob/master/explainer.md).
Some explorations here may be included in future iterations of the proposal, or
in separate proposals, but they are **not** part of the current proposal.

## Alternatives (or possible future supplements) to the current proposal

### New window placement methods or overloads of existing methods

`window.open()` is generally poorly regarded for its string-based approach to
specifying features, which have been deprecated over the years. Additionally,
current placement information and methods offer synchronous behavior, which does
not match the asynchronous behavior of user agents and OS window managers, and
precludes async permisison prompts or queries. Further, they do not clearly nor
easily support multi-screen environments.

One alternative or supplemental proposal to extending existing signatures of
Window `open()` and `moveTo|By()` methods is to add new Window Placement methods
with improved ergonomics, asynchronous behavior, user permission affordances,
and more functionality. These could also be designed with multi-screen devices,
new window display modes, and other related functionality in mind. Overloading
existing methods may provide similar benefits, but synchronous methods likely
could not return promises for overloaded signatures.

This could help move use away from existing synchronous and permissionless APIs,
which have been prone to abuse and inconsistencies over time, and which are
currently the subject of deprecation explorations.

There are several possible options for new methods or overloads:
* Add a new async method to open windows, or extend
  [`Clients.openWindow()`](https://www.w3.org/TR/service-workers-1/#dom-clients-openwindow):
  * e.g. window.openWindow(url, {left: x, top: y, width: w, height: h, screen: s, ... });
  * Support requests for specific screens and placement within that screen
  * Support requests for window states, e.g. maximized/fullscreen
  * Support requests for window display modes, e.g. standalone, minimal-ui
  * Support requests for specific inner* or outer* `width` and `height` values
  * Support existing `window.open()` features, and/or new features
  * Support feature detection via dictionary parameter?
  * Support async permission prompts or queries and underlying implementations
  * Helps moves usage away from window.open
* Add a new async API to get/set `Window` bounds and other state:
  * window.setBounds({left: x, top: y, width: w, height: h, screen: s});
  * window.getBounds() returns a Promise for corresponding information
  * Parallels proposed dictionaries for window opening above
  * Support states, display modes, etc. here or with parallel APIs
  * Support async permission prompts or queries and underlying implementations
  * Support feature detection via dictionary parameter?
  * Helps moves usage away from window.moveTo/By, screenX/Y, outerWidth/Height
* Overload `window.open()` with a dictionary parameter, modernizing the options
  available to callers and offering explicit multi-monitor support:
  * e.g. window.open(url, name, {width: w, height: h, screen: s, ... });
  * e.g. window.open(url, { name: 'name', features: '...', screen: s, ... });
  * Similar to above, but overloads cannot return promises for async behavior
* Overload `window.moveTo()` to accept an optional screen argument, treating the
  specified `x` and `y` coordinates as local to the target screen:
  * window.moveTo(x, y, {screen: s});
  * Similar to above, but overloads cannot return promises for async behavior

Here are some possible examples of new window placement methods:

```js
// NEW: Dictionary of requested features for opening a window.
let windowOptions = { left: 100, top: 100, outerWidth: 300, outerHeight: 500,
                      screen: targetScreen, state: 'normal',
                      display: 'standalone', name: 'foo',
                      /* TODO: consider child/modal/etc. */ };

// NEW: Async method supports permissions, async WM/browser implementations.
let myWindow1 = await window.openWindow(url, windowOptions);
// Or perhaps overloading the exisiting synchronous method is sufficient?
let myWindow2 = window.open(url, windowOptions);

// NEW: async methods support permissions, handle position & size cohesively.
await myWindow1.setBounds({ screen: anotherScreen, screenX: 0, screenY: 0 });
let myWindowBounds = await myWindow1.getBounds();

// NEW: Async method supports permissions, requested WM features beyond bounds.
await myWindow1.setStateAndBounds({ state: 'maximized', /*...*/ });
let myWindowStateAndBounds = await myWindow1.getStateAndBounds();
```

### Alternative names and scopes for proposed screen information methods

The proposed multi-screen info methods and events aim to offer an API shape that
naturally extends the existing single-screen `window.screen` attribute. Since
`Window` already hosts many attributes, functions and interfaces, care should be
taken not to overburden this surface. Here are some alternative shapes:

1. A `Screens` Web IDL namespace encompassing the proposed API and future
additions. For now, the new API surface may not warrant encapsulation. Using a
namespace would be preferable to a non-constructable class or interface.

2. Using the `WindowOrWorkerGlobalScope` mixin would expose the proposed API in
both `Window` and `Worker` execution contexts, extending access to service
workers, which may aid potential future work, like opening windows from service
worker launch events. This would not reduce any burden on the Window surface.

3. The [`Navigator`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator)
object could be a good potential location for the proposed API, as connected
screens could be considered an aspect of the user agent's environment.
[`WorkerNavigator`](https://developer.mozilla.org/en-US/docs/Web/API/WorkerNavigator)
could also implement the API to extend support to service workers, like above.

```js
async () => {
  // 1 & 2: Screens namespace on Window[OrWorkerGlobalScope]
  //    like: WindowOrWorkerGlobalScope.caches.keys() or
  //          WindowOrWorkerGlobalScope.indexedDB.databases()
  // Note: A `screen` namespace may conflict with the existing window.screen.
  const screensV1 = await self.screens.getScreens();
  const isMultiScreenV1 = await self.screens.isMultiScreen();
  self.screens.onscreenschange = () => { ... }

  // 3: Screens namespace (or perhaps directly) on [Worker]Navigator
  //    like: Navigator.bluetooth.requestDevice() (or Navigator.getVRDisplays())
  const screensV2 = await navigator.screens.getScreens();
  const isMultiScreenV2 = await navigator.screens.isMultiScreen();
  navigator.screens.onscreenschange = () => { ... }

  // Alternative names for getScreens():
  foo.getScreenInfo();  // Suitable for limited info or ScreenInfo dictionaries?
  foo.getDisplays();    // Suitable for non-Screen dictionaries?
  foo.request*();       // Match some existing requestFoo() web platform APIs?
}
```

TODO: Would `onscreenschange` require a non-constructable class or similar?

### Alternative synchronous access to new screen information

The leading proposal is an asynchronous interface. Alternatively, a synchronous
API would match the existing `window.screen` API, but may require implementers
to cache more system information and may preclude permission prompts or queries.

```js
// Window (or WindowOrWorkerGlobalScope) attribute, like `window.screen`
const availableScreens = self.screens;
```

With potential privacy concerns of exposing new screen and window placement
information, new APIs should allow user agents to minimize information exposed
without user permission. Asynchronous APIs are preferable, allowing the user
agent to prompt users for permission or perform asynchronous permission model
checks while the script continues processing any logic that does not depend on
the result of the permission check.

Additionally, system window managers themselves often provide asynchronous APIs,
and this pattern allows implementers to gather the latest available information
without blocking other application processing or requiring a cache.

### Exposing new info with the existing `Screen` interface

The existing `Screen` interface seems like a natural and appropriate object for
conveying information about each connected screen, but this interface and its
attributes are already exposed synchronously via `window.screen`, which yields a
live object that updates as screen properties change, or even when the window is
moved to another screen.

This proposal aims to return a static array of static objects and an event when
any screen information changes. This seems easy for developers to reason about,
easy for browsers to implement, and follows some general advice in the
[Web Platform Design Principles](https://w3ctag.github.io/design-principles/#live-vs-static).

Alternatives considered include returning dynamic collections of live objects
(e.g. collection length changes when screens connect/disconnect, and properties
of each Screen in the collection change with device configuration changes), or
static collections of live objects (e.g. indivual Screen properties change, but
the overall collection does not change reflect newly connected screens). There
are tradeoffs to each approach, but these seem more difficult for developers to
reason about, and add significant complexity to the implementation, reducing the
likelihood of obtaining interoperable implementations across browsers.

The leading proposal is to introduce a new `ScreenInfo` dictionary. While it is
shaped similar to the existing `Screen` interface, it has clear and separate
behavior as a static snapshot of comprehensive information for a particular
screen. Alternatives considered include exposing a `Screen` interface for each
connected screen (via `getScreens()` or `window.screens`) with:

1. New synchronously accessible `Screen` properties exposing additional info.
   Since user agents could not prompt users or make async permission model
   queries amid property access, the new properties could resolve to `undefined`
   or placeholder values if permission is denied, to mitigate privacy concerns.
   This would likely be confusing and atypical web platform behavior, and could
   conflict with existing non-standard `Screen` property behaviors.

2. New asynchronous methods on `Screen` exposing additional info. This would use
   more typical Promise patterns to support async permission prompts or queries,
   but introduces an inconsistency in how Screen information is exposed.

3. New asynchronous methods on another interface for additional `Screen` info.
   This is similar to (2), but perhaps even less ergonomic.

```js
// 1: Properties are undefined or placeholders before access is granted.
value = window.screen.newProperty;  // Resolves to `undefined` or <placeholder>.
await window.getScreens();          // Requests access, which is granted.
value = window.screen.newProperty;  // Resolves to the actual value.

// 2: Add asynchronous methods on `Screen` that may prompt for access:
value = await window.screen.getNewProperty();
value = await window.screen.getDictionaryWithNewProperties();

// 3: Add asynchronous methods on a new `Screens` interface:
value = await window.screens.getNewPropertyForScreen(myScreen);
value = await window.screens.getDictionaryWithNewPropertiesForScreen(myScreen);
```

Some of these topics are explored further in these issues:
* Should the API return static or live objects?
  ([webscreens/screen-enumeration#12](https://github.com/webscreens/screen-enumeration/issues/12))
* Should the lifetime of a Screen object be limited?
  ([webscreens/screen-enumeration#8](https://github.com/webscreens/screen-enumeration/issues/8))

### Exposing ScreenOrientation via getScreens().

Static `ScreenInfo` dictionaries seem unsuitable for exposing the methods and
event handler of the `Screen` interface's
[`Orientation`](https://w3c.github.io/screen-orientation/#screenorientation-interface)
interface member. As such, it seems reasonable to provide static snapshots of
the orientation type and angle in lieu of an `Orientation` interface object.

Alternatively, `ScreenInfo` objects could provide an `Orientation` interface
object but leave the methods and event handler inoperative; which would be
strange and still require callers to use `window.screen` for this functionality.

Yet another option would be to support cross-screen handlers, locks, etc., but
there is no known use cases for this functionality.

### Comparisons of `ScreenInfo` objects with the existing `Screen` interface

Given the difference of static `ScreenInfo` dictionaries and the live `Screen`
interface currently exposed via `window.screen`, there should be reliable ways
to compare the two, in order to get additional info for the current screen, or
determine which `ScreenInfo` is for the screen currently hosting the window.

Unfortunately, language limitations prohibit a staightforward direct comparison
of `ScreenInfo` objects with `window.screen`, like:

```js
// Find additional information about the current screen via getScreens().
const myScreen = (await getScreens()).find((s)=>{return s === window.screen;});

// Find a Screen from getScreens that is not the current screen.
const otherScreen = (await getScreens()).find((s)=>{return s != window.screen;});
```

So, it may prove useful to faciliate comparisons with one of these approaches:

```js
// 1. Expose an id on the `Screen` interface, like the `ScreenInfo` id.
const myScreen = (await getScreens()).find((s)=>{return s.id === window.screen.id;});
const otherScreen = (await getScreens()).find((s)=>{return s.id != window.screen.id;});

// 2. Denote the `ScreenInfo` hosting the window via `current` or similar.
const myScreen = (await getScreens()).find((s)=>{return s.current;});
const otherScreen = (await getScreens()).find((s)=>{return !s.current;});

// 3. Expose a separate way to get the window's current ScreenInfo.
const myScreen = await getCurrentScreen();
const otherScreen = (await getScreens()).find((s)=>{return s != myScreen;});
```

### Using `Screen` to represent the entire screen space.

Representing the entire combined screen space with the existing `Screen`
interface is inadvisable, as it would come with many complications, for example:
- The union of separate physical display bounds may be an irregular shape,
  comprised of rectangles with different sizes and un-aligned positions. This
  cannot be adequately represented by the current `Screen` interface.
- The set of connected physical displays may have different `Screen` properties.
- Access to multi-screen information could not be easily gated by permission.

### Using cross-screen coordinates or per-screen coordinates

A common convention of most window managers is to use the top-left corner of the
system's primary screen as the origin of the coordinate system used to position
other screens in a two-dimensional plane, relative to the primary screen. In
other cases, an arbitrary point may be used as the multi-screen coordinate space
origin, and all screens are placed relative to that point.

Standardized aspects of the `Window` interface generally follow this pattern.
The [`screenX`](https://drafts.csswg.org/cssom-view/#dom-window-screenx) and
[`screenY`](https://drafts.csswg.org/cssom-view/#dom-window-screeny) properties
are given "<i>relative to the origin of the
[Web-exposed screen area](https://drafts.csswg.org/cssom-view/#web-exposed-screen-area)</i>",
which is defined in terms of a singular output device, and without context for
multi-screen environments. However, in practice, these values are relative to
the device's multi-screen origin, which may not be the origin of the screen
hosting the content `Window`.

Similarly, [`moveTo()`](https://drafts.csswg.org/cssom-view/#dom-window-moveto)
specifies coordinates (x, y) "<i>relative to the top left corner of the output
device</i>", which does not account for multiple possible output devices. And
[`open()`](https://drafts.csswg.org/cssom-view/#the-features-argument-to-the-open()-method)
describes optional user-agent-defined clamping and coordinates relative to the
["Web-exposed \[available\] screen area"](https://drafts.csswg.org/cssom-view/#web-exposed-screen-information),
both of which are defined in terms of a singlular output device. Again, in
practice, these values are typically taken to be relative to the device's
multi-screen origin.

Unstandardized, but common properties of the
[`Screen`](https://developer.mozilla.org/en-US/docs/Web/API/Screen) interface
also already follow this pattern in many user agents. The values of `top`,
`left`, `availTop`, and `availLeft` are generally given in coordinates relative
to the device's multi-screen origin in practice.

While the proposal suggests using cross-screen coordinates in existing APIs,
per-screen scaling factors of the window manager (and per-document CSS pixel
differences?) may pose difficulties for reliably specifying these coordinates.

Further, cross-screen coordinates exposed without permission via existing API
surfaces (e.g. `window.screenX|Y`) pose a privacy concern, but the proposed new
permission could be used to gate access to most window placement and
cross-screen information.

Overall, it may be reasonable to pursue API shapes that employ per-screen
coordinates with a specific target screen or implicit assumptions regarding the
current screen, but the proposal suggests that the least invasive approach of
using cross-screen coordinates in existing APIs is more likely to match the
existing behaviors of most user agents and the working assumptions of existing
web applications.

Existing specifications have yielded unreliable APIs with inconsistent
implementation behaviors. Some implementations clamp the requested window bounds
to be within the same `Screen` as the host `Window`, while others do not.

TODO: Provide a comprehensive description of current behaviors.

Some proposed ways to improve the existing specifications are:
* Refine existing specification language for screen information, coordinate
  systems, clamping behavior, etc.
  * Define screen information in the context of multi-screen environments
  * Specify coordinates relative to a specific screen or multi-screen origin
  * Constrain the allowed user-agent-defined clamping for `Window` methods
  * Clarify cross-screen coordinates when multiple scale factors are involved
* Encourage implementers to use reasonable and compatible cross-screen behavior
  * Foster broader adoption of interoperable cross-screen behaviors without
    making spec changes
  * Possibly add non-normative language to specs regarding multi-screen support

### Specifying screens with id strings or `ScreenInfo` objects

The proposed method for requesting element fullscreen on a specific screens is
to specify the ScreenInfo object in the fullscreenOptions dictionary. It may be
reasonable to supply the id string of the ScreenInfo object instead. There are
minor tradeoffs with each option, but neither approach is a clear favorite:

- Sites are more likely to validate availability when supplying an object?
- Sites could store ids more easily than dictionaries?
- Browsers could optionally validate a dictionary before handling the request?

### Extend APIs to control window state, display modes, etc.

For reference, the space of windows states considered includes:
* `normal`: Normal 'restored' state (a framed window not in another state)
* `fullscreen`: Content fills the display without a frame
* `maximized`: Framed content occupies the available screen space
* `minimized`: Window is hidden and can be re-shown through OS controls
* `snapLeft|Right`: Framed content occupies half of available screen space

Window [focus](https://developer.mozilla.org/en-US/docs/Web/API/Window/focus) is
a related concept that should be considered in tandem with window state APIs.

Some possible ways that window state information and controls might be exposed:
* Show a new window with a specific state
  * Restore deprecated 'fullscreen="yes"' window.open() feature (w/permission?)
    * Apparently deprecated for abuse, poor ergonomics, and lack of support
    * Making a user-granted permission a prerequisite for this feature may help
  * Extend window.open() feature support for 'maximized="yes"' (w/permission?)
    * This may suffer from similar drawbacks as 'fullscreen="yes"' without care
  * Extend new openWindow method dictionary parameters with a `state` member
* Query or change an existing window's state
  * Support [window.minimize()](https://developer.mozilla.org/en-US/docs/Web/API/Window/minimize)
    and add similar methods to get/set individual window states
  * Add new methods to get or set the self/child/opener window state value
  * Support additional window.focus() scenarios (self, opener, etc.)
  * Support explicit z-ordering, such as an `"alwaysOnTop"` window state
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
* Minimize/restore/focus associated windows per user control of a 'main' window:
  * Doctor minimizes patient case window, app minimizes associated image windows
  * Doctor selects patient case entry in a list, app restores minimized windows
* Web application offers settings to show or hide minimal-ui native controls
* Video conferencing window wishes to be [always-on-top](https://github.com/webscreens/window-placement/issues/10)

There are open questions around the value and uses cases here:
* Need additional attestation of compelling use cases from developers
* Need to assess the risks and mitigations versus the utility offered
* Consider creation and management of dependent or 'child' window types
* Some capabilities are may warrant additional permission requirements, etc.

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

### Surface events on window bounds, state, or display mode changes

Currently, `Window` fires an event when content is resized:
[onresize](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onresize).
There may be value in firing events when windows move, when the state changes,
or when the display mode changes:
* Add `"onmove"`, `"onwindowdrag"`, and `"onwindowdrop"` Window events
* Add `"onwindowstate"` Window event for state changes
* Add `"onwindowdisplaymode"` Window event for display mode changes

This would allow sites to observe window placement, state, and display mode
changes, useful for recognizing, persisting, and restoring user preferences for
specific windows. This would also be useful in scenarios where the relative
placement of windows might inform automated placement of accompanying windows
(e.g. a grid of windows or 'child' window behavior).

### Support multiple fullscreen elements from a single document

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

### Extend placement affordances on transient user activation

Currently, the transient user activation of a button click is consumed when
opening a window. That complicates or blocks scenarios that might wish to also
request fullscreen from the event handler for a single user click. Relatedly,
most browsers allow sites to open multiple popup windows with a single click if
they have user permission.

Allowing new behaviors would solve some valuable use cases, for example:
* Requesting fullscreen and opening a separate new window:
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
  const slides = window.open(`./slides`, ``, features);
  // NEW: `screen` on `fullscreenOptions` for `requestFullscreen()`.
  slides.document.body.requestFullscreen({ screen: externalScreen });
  ```

### Allow placements that span two or more screens

With the introduction of new foldable devices, and with some affordances of
having a single content window in multi-screen environments, it may become more
common for windows to span multiple displays. This may be worth considering as
new multi-screen aware Window Placement APIs are developed.

### Support dependent or 'child' window types

Dependent or child windows are useful in certain native application contexts:
* Creative apps (image/audio/video) with floating palettes, previews, etc.
* Medical applications with separate windows for case reports and images

These windows might be grouped with their parent window is some OS entrypoints,
like taskbars and window switchers, and might move and minimize/restore in
tandem with the parent window.

```js
// Open a dependent/child window that the OS/browser moves with a parent window.
const palette = window.open("/palette", "palette", "dependent=true");
// Alternately use a proposed move event to move a companion window in tandem.
window.addEventListener("move", e => { palette.moveBy(e.deltaX, e.deltaY); });
```

### New `ScreenInfo` properties to consider as use cases arise

* `accelerometer`: True if the display has an accelerometer.
  * May be useful for showing immersive controls (e.g. game steering wheel).
  * Not web-exposed
* `dpi`: The display density as the number of pixels per inch.
  * May be useful for presenting content with tailored physical scale factors.
  * Web-exposed for the window's current screen via
    [`Window.devicePixelRatio`](https://drafts.csswg.org/cssom-view/#dom-window-devicepixelratio)
* `subpixelOrder`: The order/orientation of this display's subpixels.
  * May be useful for adapting content presentation for some display technology.
  * Not web-exposed
* `interlaced`: True if the display's mode is interlaced.
  * May be useful for adapting content presentation for some display technology.
  * Not web-exposed, but available via the Chrome Apps
    [`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo)
* `refreshRate`: The display's refresh rate in hertz.
  * May be useful for adapting content presentation for some display technology.
  * Not web-exposed, but available via the Chrome Apps
    [`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo)
* `overscan`: The display's insets within its screen's bounds.
  * May be useful for adapting content presentation for some display technology.
  * Not web-exposed, but available via the Chrome Apps
    [`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo)
* `mirrored`: True if the display is mirrored to another display.
  * May be useful for recognizing when a laptop is mirrored to a projector, etc.
  * Not web-exposed, but available via the Chrome Apps
    [`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo)
* `hidden`: True if the display is not visible (e.g. closed laptop).
  * May be useful for recognizing when displays may be active but not visible.
  * Not web-exposed

### New `Window` properties to consider exposing

* The window state: (e.g. maximized, normal/restored, minimized, side-snapped)
* The window type or display (e.g. normal/tab, popup, standalone application)
* Events on changes: (e.g. onmove, onwindowdrag, onwindowdrop, onwindowstate)
* Enumerating the list of existing windows opened for a given worker/origin

### Allowing sites to enumerate their windows

This may be useful as web applications support URL handling or launch events,
perhaps through service worker execution contexts.
* Add `client.Enumerate()` to list existing windows from a Service Worker

### User agent screen pickers and limited power APIs

Alternative API shapes giving less power to sites were considered, but generally
offer poorer experiences for users and developers. For example,
[webscreens/screen-enumeration#23](https://github.com/webscreens/screen-enumeration/issues/23)
asks whether a picker-style API could be simpler, less powerful, and still
fulfill the users cases. While these are highly desirable traits of web platform
APIs, some stated use cases would be cumbersome for users and developers, if not
impossible, if a user-agent screen picker or more strictly limited APIs were the
only tools available.

Here are some options explored, outside the separate topic of "Supporting
requests for limited information", which is considered below:

* Show multi-screen users a user-agent controlled screen-picker on window.open()
  or element.requestFullscreen(), perhaps only when a site suggests that the
  content is intended for certain multi-screen use (e.g. presentation material)
  or most suited to a particular type of screens (e.g. HDR, relative position).
  * This approach limits cross-screen information initially exposed to sites,
    but it still exposes the resulting window.screen info.
  * This could be as cumbersome for users as dragging windows to target screens.
  * It would be preferable in certain use cases for web applications to select
    an optimal screen based on content and available screen traits, or to
    specify a screen according to a user's preselected preferences, or past use.
  * Requests to open windows with particular bounds could not be easily tailored
    for the target screen, if that screen isn't known in advance of the request.
  * Scenarios involving presentation of content on multiple screens would
    amplify the cumbersome effects of a picker and increase the chances that
    web applications could give valuable direction over the resulting layout.
* Show multi-screen users user-agent controls to move windows between screens.
  * Again, the resulting window.screen information is likely still available.
  * This could be as cumbersome for users as dragging windows to target screens.
  * This does not allow web applications to suggest optimal initial placements.
  * This does not allow web applications to save or apply user preferences.
  * This may conflict with OS-specific paradigms for similar window controls.
* Only provide declarative APIs for web applications to place windows on the
  most appropriate screen, without exposing new info about available screens.
  * Again, the resulting window.screen information is likely still available.
  * It would be difficult to declaratively capture the full expressiveness of
    preferences that a web application might wish to convey regarding the
    content or circumstances of a fullscreen request.
* Have the user agent apply window placements based on a user's past use.
  * Again, the resulting window.screen information is likely still available.
  * It is unclear how a user would convey screen preferences without dragging
    windows or using an initial picker, and it's unclear how granular these
    preferences should be applied, or how a user might convey more nuanced or
    updated preferences.
  * This could only benefit users on subsequent uses.

### Supporting requests for limited information

It may be beneficial to extend the proposed API with a mechanism to request more
limited or granular multi-screen information. This may allow web developers and
user agents to cooperatively request and provide information required for
specific use cases, proactively reducing the fingerprintable information shared,
and potentially allowing user agents to expose more limited information without
explicit user permission prompts.

Perhaps getScreens() could request everything by default, and take an optional
dictionary parameter to request limited information. Results could be returned
in a new dictionary that could include a ScreenInfo sequence (perhaps with only
the requested values populated?), and/or ancillary members with the specific
requested information. Here are a couple examples of how that might work, and
replace the proposed `isMultiScreen()` limited information query.

```js
// Request a single bit answering the question: are multiple screens available?
// This informs the value of additional information requests (and user prompts).
let screen_info = await getScreens({multiScreen: true});
if (!screen_info.multiScreen)
  return;
// Request the number of connected screens, either returning an array of 'empty'
// Screen objects with undefined property values, or as a named member of a
// returned dictionary, eg: { multi-screen: true, count: 2, ... }.
screen_info = await getScreens({count: true});
if (screen_info.count <= 1)  // OR: |if (screen_info.screens.length <= 1)|
  return;

// Screen ids alone may suffice for some proposed Window Placement uses.
screen_info = await getScreens({id: true});
document.body.requestFullscreen(screen_info.screens[1].id);

// OR: call getScreens() again to request additional information, with only the
// requested information available via corresponding `Screen` attributes.
screen_info = await getScreens({availWidth: true, colorDepth: true});
// Use availWidth and colorDepth to determine the appropriate display.
// TODO: Refine this naive example with something more realistic.
document.body.requestFullscreen(
    (screen_info.screens[1].availWidth > screen_info.screens[0].availWidth ||
      screen_info.screens[1].colorDepth > screen_info.screens[0].colorDepth)
    ? screen_info.screens[1] : screen_info.screens[0]);
}
```

## Additional topics and considerations

### Nomenclature

Existing native APIs use a variety of nomenclatures to describe the distinction
between physical display devices and the overall space composed by their virtual
arrangement. As the web platform already uses the `Screen` interface to describe
a single physical unit of rendering space, it seems natural to follow this
convention and work in terms of a multi-`Screen` display environment.

### Changes to `colorDepth` and `pixelDepth`

The [W3C Working Draft](https://www.w3.org/TR/cssom-view/#dom-screen-colordepth)
states that `Screen.colorDepth` and `Screen.pixelDepth` "must return 24" and
even explains that these "attributes are useless", but the latest
[Editor's Draft](https://drafts.csswg.org/cssom-view/#dom-screen-colordepth)
provides a more useful specification for these values. There is a clear signal
from developers that having meaningful and accurate accurate values for these
properties is useful for selecting the optimal display to present medical and
creative content.

### Relation to Presentation and Remote Playback APIs

The [Presentation](https://www.w3.org/TR/presentation-api/) and
[Remote Playback](https://www.w3.org/TR/remote-playback/) APIs provide access to
displays connected on remote devices, and to device-local secondary displays,
but they are geared towards showing a single fullscreen content window on each
external display and have other limitations regarding our described use cases.

This proposal aims to offer compelling features that complement and extend
existing Web Platform APIs. For example this proposal would offer sites the
ability to show their own list of displays to the user, open non-fullscreen
windows, limit display selection to those directly connected to controller
device (not those on remote devices), instantly show content on multiple
displays without separate user interaction steps for each display, and to swap
the displays hosting each content window without re-selecting displays.

### Handling subsequent calls to `Element.requestFullscreen()`

Subsequent calls targetting a different screen could move the fullscreen window
to another screen. This allows web applications to offer users a simple way
to swap screens used for fullscreen. For example:

```js
// Step 1: Slideshow starts fullscreen on the current screen (nothing new).
document.body.requestFullscreen();
...
// Step 2: User invokes a slideshow control setting to use another screen.
// NEW: `requestFullscreen()` supports subsequent calls with different screens.
document.body.requestFullscreen({ screen: selectedScreen });
```

Other prospective solutions (like exiting fullscreen, moving the window across
displays, and re-requesting fullscreen) may fail due to the consumption of
transient user activation or suffer from undesirable flickering of intermediate
states, not to mention the poor developer ergonomics of that process.

Note: It may not be feasible or straightforward for multiple elements in the
same document to show as fullscreen windows on separate screens simultaneously.

### Permission requirements to receive `screenschange` events

Sites that have called `isMultiScreen()` may be interested in `screenschange`
events that would result in a different `isMultiScreen()` value (e.g. a second
screen was connected or disconnected). This avoids the need for such sites to
poll `isMultiScreen()` and obviates the need for a `multiscreenchanged` event.

Sites that have called `getScreens()` may be additonally interested in
`screenschange` events that would yield updated screen information snapshots
(e.g. the display properties of a connected screen have changed).

Browser implementations may require permissions for any proposed functionality,
and may choose to require a basic permission to fulfill `isMultiscreen()` calls
and send corresponding `screenschange` events, while requiring a more strict
permission to fulfill `getScreens()` calls and send corresponding
`screenschange` events.

TODO: Explore additional permission and privacy considerations here?

### Window placement affordances around `screenschange` events

This proposal aims to allow some window placement actions on screen change
events. For example, it may be reasonable for a site to move a fullscreen window
onto a newly connected display:

```js
self.addEventListener('screenschange', function(event) {
  // Ensure the best screen is used for fullscreen on screen change events.
  // NEW: screenschange event handlers may call `requestFullscreen()`.
  if (document.fullscreenElement)
    document.body.requestFullscreen({ screen: getBestScreen() });
});
```

Since user-generated screen change events are not user activations of the site,
we suggest specifically allowing Element.requestFullscreen() when the algorithm
is triggered by a user-generated screen change. This parallels the existing
[spec](https://fullscreen.spec.whatwg.org/#dom-element-requestfullscreen)
allowance when triggered by a "user generated orientation change".

### Working with inner size versus outer size

window.open() treats supplied width and height feature string values as the
inner/content size, while window.resizeTo() treats supplied width and height
values as outer/window sizes. Ideally developers could supply either inner or
outer dimensions for either method.

There is likely no meaningful reason for a site to supply the inner location of
a window, (i.e. the top and left screen coordinates of the content, inset inside
the window frame).

### Window placement limitations

Changes allowing cross-screen window placements should still be subject to
reasonable limitations, and some may be left to the discretion of implementers.

Implementers generally restrict window movement to certain windows types, (e.g.
standalone web app windows and popup windows, not tabbed browser windows) and
only under certain circumstances (e.g. secure contexts, with transient user
activation, maybe limiting the number or frequency of interactions, etc.).

Window placements could be clamped to the available bounds of a single target
screen, or perhaps allowed to span multiple screens. It seems reasonable for
implementers to restrict window placements that would make some portion of
the window appear outsides the aggregate available screen bounds.

It may be valuable to provide clearly specified behaviors for some cases, or to
offer non-normative guidance for how such cases could be handled.

### Respecting intents around user-agent-defined behaviors

Reasonable caution should be exercised when changing implementation-specific
behavior of existing Web Platform APIs, to avoid breaking existing users. The
leading proposal aims to align behavior around the existing behavior of some
implementers, and honor perceived intent.

Still, it may be prudent for implementers to retain elements of old behavior
that sites might rely upon. For example, if developers expect window placements
to be clamped within the current `Screen` and supply bounds outside the overall
`Screen` space (e.g. "left=99999999"), then it may be reasonable to clamp
window placements within the current `Screen`, rather than within the `Screen`
nearest those coordinates.

### TODO: Explore additional topics

- Multi-screen media query (no clear permission support?)
- Move ScreenInfo orientation* properties into a separate/nested dictionary?
- Should window.screen ultimately be a static snapshot, instad of a live object?
- Is ScreenOrientation access and functionality via `window.screen` appropriate?
- Cross-screen window placements may be difficult to express in CSS Pixels?
  - Use hardware pixels or DIPs in ScreenInfo, existing or new placement APIs?
  - Take Screen object/id and screen-local CSS Pixel coordinates in new APIs?

## Other miscellaneous use cases and explorations

### Financial dashboard web application

A financial dashboard opening multiple windows across multiple screens, using
the overall screen-space information to produce a grid-like window arrangement.
The application can persist and restore the layout between user sessions.

TODO: Update this outdated exploration

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
