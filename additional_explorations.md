# Additional Explorations of Window Placement on the Web

This document explores alternatives, considerations, and examples that have
helped to shape the current proposal, which is fully described in
[EXPLAINER.md](https://github.com/webscreens/window-placement/blob/master/explainer.md).
Some explorations here may be included in future iterations of the proposal, or
in separate proposals, but they are **not** part of the current proposal.

## Alternatives to the current proposal

### TODO: Explore these alternatives:

- multi-screen media query (no clear path for user-facing permission prompt support)
- Browser-controlled multi-screen placement interstitials
- Move ScreenInfo orientation* properties into a separate/nested dictionary
- screen picker, limited apis
- comparing ScreenInfo objects with window.screen, or exposing the current screen

### Alternative new methods

One alternative to extending Window `open()` and `moveTo|By()` methods is to
introduce entirely new Window Placement methods. These could be built to solve
frequent requests of modern web applications; offering better ergonomics,
asynchronous behavior, user permission affordances, and more functionality.
These could also be designed with multi-screen devices, new window display
modes, and related functionality in mind.

This could enable further deprecation of existing synchronous and permissionless
APIs, (e.g. `window.outerWidth|outerHeight`, `window.open` and features), which
have been prone to abuse and inconsistencies over time, and which are currently
the subject of various privacy and deprecation explorations.

Of course, aspects of new methods could prove undesirable in the long term, as
we've found with existing APIs, and it may be difficult to deprecate old APIs,
so this approach deserves caution. Here are some possible shapes of new window
placement methods:

TODO: Provide an example of new openWindow, setBounds, setState, get*, etc.

### Alternative names and scopes for proposed methods

Besides the proposed shape of new methods and events on `Window`, some
alternative names and locations are demonstrated below.

A `Screens` Web IDL namespace could provide a shared location for `getScreens()`
and related functionality that may be added in the future, but at this time, the
new API surface may be too limited to motivate that approach. Using a namespace
would be preferable to a non-constructable class or interface.

```js
async () => {
  // 1: Screens namespace on Window[OrWorkerGlobalScope], or [Worker]Navigator,
  //    like: WindowOrWorkerGlobalScope.caches.keys(),
  //          WindowOrWorkerGlobalScope.indexedDB.databases(), or
  //          Navigator.bluetooth.requestDevice()
  // Note: A `screen` namespace may conflict with the existing window.screen.
  const screensV1 = await self.screens.get();
  const screensV2 = await navigator.screens.get();

  // 2: Directly on [Worker]Navigator, like: Navigator.getVRDisplays()
  const screensV3 = await navigator.getScreens();

  // Alternative names for getScreens():
  foo.getScreenInfo();  // Suitable for limited info or non-Screen dictionaries?
  foo.getDisplays();    // Suitable for non-Screen dictionaries?
  foo.request*();       // Match some existing requestFoo() web platform APIs?
}
```

### Alternative synchronous access pattern

The leading proposal is an asynchronous interface. Alternatively, a synchronous
API would match the existing `window.screen` API, but may require implementers
to cache system information that may not otherwise be required. It would also
require script execution to halt while a permission model is queried or the user is prompted.

```js
// Window or WindowOrWorkerGlobalScope attribute, like `window.screen`
const avialableScreens = self.screens;
```

### Alternative property access or a new `Display` container class

The existing [`Screen`][1] object seems like a natural and appropriate container
for conveying information about the connected display devices, but using the
`Screen` interface to convey **new** properties poses a potential privacy
concern with regard to fingerprinting, since the properties of the window's
current display would be exposed synchronously by the `window.screen` API. User
agents would not be able to gate access to these new properties on an
asynchronously-queried permission model.

Alternatively, the new properties could be accessed by new asynchronous methods
on the `Screen` or `ScreenManager` interfaces, or resolve to undefined if access
has not been granted:

```js
// 1: Add an asynchronous method on `Screen`:
const value = window.screen.getNewProperty();

// 2: Add an asynchronous method on `ScreenManager`:
const value = navigator.screen.getNewPropertyForScreen(window.screen);

// 3: Properties are undefined before access is granted.
console.log(screen.newProperty);  // Resolves to `undefined`.
navigator.screen.getScreens();    // Requests access, which is granted.
console.log(screen.newProperty);  // Resolves to the actual value.
```

Yet another alternative is introducing a new `Display` interface as a container
class that supercedes or parallels the `Screen` interface. This would duplicate
some properties but ensure that new, potentially privacy-sensitive properties
would only be exposed asynchronously, after potentially checking for permission.
This may cause some confusion or difficulty correlating the existing `Screen`
object with a new given `Display` object.

### Alternatively using `Screen` to represent the entire screen space.

Representing the entire combined screen space with the existing `Screen`
interface is inadvisable, as it would come with many complications, for example:
* The union of separate physical display bounds may be an irregular shape,
  comprised of rectangles with different sizes and un-aligned positions. This
  cannot be adequately represented by the current `Screen` interface.
* The set of connected physical displays may have different `Screen` properties.





TODO: New `Screen` properties to consider as use cases arise.
* `Screen.accelerometer`: True if the display has an accelerometer.
  * May be useful for showing immersive controls (e.g. game steering wheel).
* `Screen.dpi`: The display density as the number of pixels per inch.
  * May be useful for presenting content with tailored physical scale factors.
* `Screen.subpixelOrder`: The order/orientation of this display's subpixels.
  * May be useful for adapting content presentation for some display technology.
* `Screen.interlaced`: True if the display's mode is interlaced.
  * May be useful for adapting content presentation for some display technology.
* `Screen.refreshRate`: The display's refresh rate in hertz.
  * May be useful for adapting content presentation for some display technology.
* `Screen.overscan`: The display's insets within its screen's bounds.
  * May be useful for adapting content presentation for some display technology.
* `Screen.hidden`: True if the display is not visible (e.g. closed laptop).
  * May be useful for recognizing when displays may be active but not visible.
* `Screen.mirrored`: True if the display is mirrored to another display.
  * May be useful for recognizing when a laptop is mirrored to a projector, etc.

TODO: Inspiration for the shape of this API comes from the existing `Window.screen`
attribute location, with expanded access to worker contexts. Similar APIs exist,
like `navigator.languages` and `self.addEventListener("onlanguagechange", ...)`.

TODO: See the Alternative Proposals section for some other possible API shapes.

## Additional considerations

### Using screen coordinates relative to the primary screen in existing APIs
This aspect of the proposal aims to support cross-screen window coordinates
within the existing web-exposed API.

One non-trivial complication is the meaning of per-screen and per-document scale
factors on cross-screen CSS pixel values used to set window placements, which
ultimately 

### Synchronicity

One advantage of asynchronous APIs is that they are non-blocking. With potential
privacy concerns with screen enumeration, it is possible that the API should
only expose screen information if the user has granted permission. In this case,
asynchronicity is preferable as it allows the script to continue processing any
logic that does not depend on the result of the permission check.

Additionally, system window managers themselves often provide asynchronous APIs,
and this pattern would allow implementers to gather the latest available
information without blocking other application processing or requiring a cache.

### Nomenclature

Existing native APIs use a variety of nomenclatures to describe the distinction
between physical display devices and the overall space composed by their virtual
arrangement. As the web platform already uses the [`Screen`][1] interface to
describe a single physical unit of rendering space, it seems natural to follow
this convention and work in terms of a multi-`Screen` display environment.

### Multi-Screen Coordinates

By common convention, the top-left corner of the system's primary display
defines the origin of the coordinate system used to position other displays
in a two-dimensional plane, relative to the primary display.

Although it is not standardized, the [`Screen`][1] interface already follows
this same pattern for multi-screen environments in practice. The unstandardized
properties `top`, `left`, `availTop`, and `availLeft` match the system's virtual
arrangement of separate physical displays in practice. So, the `Screen` object
for the primary display has `top` and/or `left` values of zero, while `Screen`
objects for secondary displays have non-zero `top` and/or `left` values,
denoting their placement relative to the primary display.

Unstandardized aspects of the [`Window`][2] interface follow the same pattern.
The [`screenX`](https://developer.mozilla.org/en-US/docs/Web/API/Window/screenX)
and [`screenY`](https://developer.mozilla.org/en-US/docs/Web/API/Window/screenY)
properties are given relative to the origin of the primary display, which is not
necessarily the host of the content `Window`. Further, parameters passed into
[`moveTo()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/moveTo) are
taken to be in the same coordinate space, relative to the primary display.

It may be beneficial to actually standardize this pattern, or to provide some
non-normative notes encouraging implementers to follow this common convention.

### Scope: `Window`, `WindowOrWorkerGlobalScope`, `Navigator`/`WorkerNavigator`

Taking inspiration from existing Web APIs, there are a few places where the
proposed API could reasonably live.

1. In the `Window` interface, making multi-screen info accessible to execution
contexts using the existing single-screen `window.screen` attribute. Those
familiar with `window.screen` might anticipate a similar access pattern for
information about multi-screen environments. It should be noted that `Window`
already hosts many attributes, functions and interfaces, so care should be taken
not to overburden this surface.

2. The `WindowOrWorkerGlobalScope` mixin, implemented by `Window` and
`WorkerGlobalScope` interfaces, could also be an intuitive host this API. This
offers access to both `Window` and `Worker` execution contexts, extending the
suggestion above with additional access for service workers, which may be
valuable for certain aspects of the the [Window Placement API proposal][3].

2. The [`Navigator`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator)
object nested beneath the global scope may be a good potential location for this
API, as the set of connected displays could be reasonably considered an aspect
of the user agent's environment. In order to support the API in service workers,
[`WorkerNavigator`](https://developer.mozilla.org/en-US/docs/Web/API/WorkerNavigator)
would also need to implement the API.

### Changes to `colorDepth` and `pixelDepth`

The [W3C Working Draft](https://www.w3.org/TR/cssom-view/#dom-screen-colordepth)
states that `Screen.colorDepth` and `Screen.pixelDepth` "must return 24" and
even explains that these "attributes are useless", but the latest
[Editor’s Draft](https://drafts.csswg.org/cssom-view/#dom-screen-colordepth)
provides a more useful specification for these values. There is a clear signal
from developers that having meaningful and accurate accurate values for these
properties is useful for selecting the optimal display to present medical and
creative content.

### Compatibility with the existing window.screen object

This proposal aims to provide a high degree of compatibility between objects
returned by the getScreens() API and the existing synchronously-accessed
`window.screen` object. Ideally, sites should be able to compare Screens objects
without concern for their origin, enabling a very basic and useful check like:

```js
// Find a Screen from getScreens that is not the current screen.
const otherScreen = (await getScreens()).find((s)=>{return s != window.screen;});
```

There are some options around exposing new properties for each access pattern:
* New properties are only exposed on getScreens() objects
  * window.screen yields undefined values for new properties
  * Comparison may just regard underlying devices, not property values
* New properties are exposed on getScreens() objects and window.screen
  * This requires synchronous access to the new properties on window.screen
  * How to support permission requirements of synchronously exposing new info

The existing Screen (via window.screen) is a live object. A cached Screen object
(eg. `const myScreen = window.screen`) returns updated values after screen changes,
and even updates when the window is moved across displays.

This proposal aims to return a static array of static objects and an event when
any screen information changes. This seems easy for developers to reason about,
easy for browsers to implement, and follows some general advice in the
[Web Platform Design Principles](https://w3ctag.github.io/design-principles/#live-vs-static).

Alternative approaches include returning dynamic collections of live objects
(eg. collection length changes when screens connect/disconnect, and properties
of each Screen in the collection change with device configuration changes), or
static collections of live objects (eg. indivual Screen properties change, but
the overall collection does not change reflect newly connected screens). There
are tradeoffs to each approach, but these seem generally less desirable.

The difference of static screen information snapshots from getScreens() and the
live Screen object from window.screen yields some compatibility questions:
* How static getScreens() objects handle the methods and event handler of
  [screen.orientation](https://developer.mozilla.org/en-US/docs/Web/API/ScreenOrientation)
  * Leave these inoperative; callers must access window.screen
  * Support these APIs for cross-screen handlers, locks, etc.
* Should ScreenOrientation have another access method?
* Should window.screen ultimately be a static snapshot, instad of a live object?

Some of these topics are explored further in these open issues:
* Should the API return static or live objects?
  ([#12](https://github.com/webscreens/screen-enumeration/issues/12))
* Should the lifetime of a Screen object be limited?
  ([#8](https://github.com/webscreens/screen-enumeration/issues/8))

### Requests for limited information

One particularly valuable example of limited information is proposed above, to
answer the basic question "are there multiple screens?" via `isMultiScreen()`.

It may be beneficial to extend the proposed API with a mechanism to request more
limited or granular multi-screen information. This may allow web developers and
user agents to cooperatively request and provide information required for
specific use cases, proactively reducing the fingerprintable information shared,
and potentially allowing user agents to expose more limited information without
explicit user permission prompts (eg. a single multi-screen boolean flag).

One possible approach is that getScreens() could request everything by default,
and take an optional parameter to request limited information. Partial results
could be returned as a Screen array with only the requested values populated, or
with dictionaries of named values including the screen array.

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
if (screen_info.count <= 1)  // OR: |if (screen_info.length <= 1)|
  return;

// An empty Screen object may suffice for some proposed Window Placement uses.
document.body.requestFullscreen(screens[1]);

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

### Relation to Presentation and Remote Playback APIs

The [Presentation](https://www.w3.org/TR/presentation-api/) and
[Remote Playback](https://www.w3.org/TR/remote-playback/) APIs provide access to
displays connected on remote devices, and to device-local secondary displays,
but they are geared towards showing a single fullscreen content window on each
external display and have other limitations regarding our described use cases.

This proposal and the associated [Window Placement API][2] proposal aim to offer
compelling features that complement and extend existing Web Platform APIs. For
example this proposal would offer sites the ability to show their own list of
displays to the user, open non-fullscreen windows, limit display selection to
those directly connected to controller device (not those on remote devices),
instantly show content on multiple displays without separate user interaction
steps for each display, and to swap the displays hosting each content window
without re-selecting displays.

### Adding a screen id instead of a ScreenInfo to fullscreenOptions

It may be reasonable to support element fullscreen requests for specific screens
with an id or a ScreenInfo object. There are minor tradeoffs with each option:

TODO: Explore these tradeoffs in more detail?

- Sites could store an id easier than a dictionary?
- Browsers could optionally validate a dictionary before handling the request?

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
This topic is discussed futher in [additional_explorations.md](https://github.com/webscreens/window-placement/blob/master/additional_explorations.md).

### Permission requirements to receive `screenschange` events

TODO: Explore additional permission and privacy considerations here? 

Sites that have called `isMultiScreen()` may be interested in `screenschange`
events that would result in a different `isMultiScreen()` value (e.g. a second
screen was connected or disconnected). This avoids the need for such sites to
poll `isMultiScreen()` and obviates the need for a `multiscreenchanged` event.

Sites that have called `getScreens()` may be additonally interested in
`screenschange` events that would yield updated screen informtion snapshots
(e.g. the display properties of a connected screen have changed).

Browser implementations may require permissions for any proposed functionality,
and may choose to require a basic permission to fulfill `isMultiscreen()` calls
and send corresponding `screenschange` events, while requiring a more strict
permission to fulfill `getScreens()` calls and send corresponding
`screenschange` events.

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

### Improving existing API specifications and implementations

As alluded to above, the existing `Window` specifications do not provide
reliable mechanisms for multi-screen placement. Notably,
[`moveTo()`](https://www.w3.org/TR/cssom-view-1/#dom-window-moveto)
describes coordinates (x, y) as "relative to the top left corner of the output
device", which does not account for multiple possible output devices. Similarly,
[`open()`](https://drafts.csswg.org/cssom-view/#the-features-argument-to-the-open()-method)
describes optional user-agent-defined clamping and coordinates relative to the
"Web-exposed \[available\] screen area", which also describes areas in terms of
a single output device. See
[CSSOM View Module 2.3. Web-exposed screen information](https://drafts.csswg.org/cssom-view/#web-exposed-screen-information).
This has yielded an unreliable API with inconsistent implementation behaviors.
Some implementations clamp the requested window bounds to be within the
same `Screen` as the host `Window`, while others do not.

TODO: Provide a comprehensive description of current inconsistent behaviors.

Proposed ways to improve the existing surface area are:
* Refine existing specification language for screen information, coordinate
  systems, clamping behavior, etc.
  * Define screen information in the context of multi-screen environments
  * Specify `Window` coordinates as relative to the primary (or current) screen
  * Constrain the allowed user-agent-defined clamping for `Window` methods
* Encourage implementers to use reasonable and compatible cross-screen behavior
  * Foster broader adoption of interoperable cross-screen behaviors without
    making spec changes
  * Possibly add non-normative language to specs regarding multi-screen support

### Working with inner size versus outer size

window.open() treats supplied width and height feature string values as the
inner/content size, while window.resizeTo() treats supplied width and height
values as outer/window sizes.

TODO: This is strange, we should probably allow developers to supply either
inner or outer values for either operation...

There is likely no meaningful reason for a site to supply the inner location of
a window, (i.e. the top and left screen coordinates of the content, inset inside
the window frame).

### Working with CSS Pixels or other units

TODO: CSS Pixels may not be the most straightforward way to place windows...
TODO: List alternatives:
  - supply hardware pixels or DIPs in Screen info?
  - take hardware pixels or DIPs in window.open/moveTo/moveBy or new APIs?
  - take Screen object/id and screen-local CSS pixel coordinates in new apis?

## Additional Thoughts

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

It may be valuable to provide clearly specifided behaviors for some cases, or to
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

### Changes to `colorDepth` and `pixelDepth`

The [W3C Working Draft](https://www.w3.org/TR/cssom-view/#dom-screen-colordepth)
states that `Screen.colorDepth` and `Screen.pixelDepth` "must return 24" and
even says these "attributes are useless", but the latest
[Editor’s Draft](https://drafts.csswg.org/cssom-view/#dom-screen-colordepth)
provides a more useful specification for these values. Exposing accurate values
for these properties is useful for placing medical and creative content on the
optimal screen.

## Possible future goals of the [current proposal](https://github.com/webscreens/window-placement/blob/master/explainer.md)

Future goals may provide additional capabilities and more ergonomic APIs for web
applications to manage windows. See explorations of these tentative goals below.
* Offer more ergonomic APIs
* Extend window state and display mode APIs
* Surface events on window bounds, state, or display mode changes
* Support multiple fullscreen elements from a single document
* Extend placement affordances on transient user activation
* Allow placements that span two or more screens
* Support dependent or 'child' window types
* Allow sites to enumerate their windows

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
* Overload `window.moveTo()` to accept an optional screen argument, treating the
  specified `x` and `y` coordinates as local to the target screen:
  * window.moveTo(x, y, {screen: targetScreen});
* Add a new async API to open windows, maybe extend (and expose on Window)
  [`Clients.openWindow()`](https://www.w3.org/TR/service-workers-1/#dom-clients-openwindow)
  via a `windowOptions` dictionary parameter with multi-screen support:
  * window.openWindow(url, {left: x, top: y, width: w, height: h, screen: s});
    * Coordinates specified relative to the target screen OR:
    * Nix `screen` and specify coordinates relative to the primary screen
  * Support requests for window states, eg. maximized/fullscreen?
  * Support requests for window display modes, eg. standalone, minimal-ui?
  * Should `width` and `height` be inner* or outer* values, or accept either?
  * Support existing `window.open()` features, and/or new features?
  * Async allows permission requests; dictionary allows feature detection
* Add a new async API to get/set `Window` bounds:
  * window.setBounds({left: x, top: y, width: w, height: h, screen: s});
  * window.getBounds() returns a Promise for corresponding information
  * Parallels proposed dictionaries for window opening above
  * Support states, display modes, etc. here or with parallel APIs, like below?
  * Async allows permission requests; dictionary allows feature detection

## Extend window state and display mode APIs

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
  * Extend new open window function dictionary parameters with a `state` member
* Query or change an existing window's state
  * Support [window.minimize()](https://developer.mozilla.org/en-US/docs/Web/API/Window/minimize)
    and add similar methods to get/set individual window states
  * Add new methods to get or set the self/child/opener window state value
  * Support additional window.focus() scenarios (self, opener, etc.)
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
* Minimize/restore/focus associated windows per user control of a 'main' window:
  * Doctor minimizes patient case window, app minimizes associated image windows
  * Doctor selects patient case entry in a list, app restores minimized windows
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

## Surface events on window bounds, state, or display mode changes

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
(eg. a grid of windows or 'child' window behavior).

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

## Extend placement affordances on transient user activation

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
  const slides = window.open(`./slides`, ``, ``);
  // NEW: `screen` on `fullscreenOptions` for `requestFullscreen()`.
  slides.document.body.requestFullscreen({ screen: externalScreen });
  ```

## Allow placements that span two or more screens

With the introduction of new foldable devices, and with some affordances of
having a single content window in multi-screen environments, it may become more
common for windows to span multiple displays. This may be worth considering as
new multi-screen aware Window Placement APIs are developed.

## Support dependent or 'child' window types

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

## Allow sites to enumerate their windows

This may be useful as web applications support URL handling or launch events.
* Add `client.Enumerate()` to list existing windows from a Service Worker

## Other miscellaneous explorations

### TODO: Doctor examples:
TODO: For example, a doctor
editing patient records on a one screen may wish images to be opened in separate
windows on another screen with the appropriate resolution and color depth. These
windows should be resizable and not be fullscreen, Opening and  use another screen for a laptop the optimal screen for a given use case.

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
