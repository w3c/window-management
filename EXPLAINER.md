# Multi-Screen Window Placement on the Web

## Introduction

This proposal introduces incremental improvements to existing screen information
and window placement APIs, allowing web applications to be intentional about the
experiences they offer to users of multi-screen devices.

## Background

Operating systems generally allow users to connect multiple screens to a single
device and arrange them virtually to extend the overall visual workspace.

A variety of applications use platform tools to place windows in multi-screen
environments, but web application developers are limited by existing APIs, like
[`Screen`](https://drafts.csswg.org/cssom-view/#the-screen-interface) and
[`Window`](https://drafts.csswg.org/cssom-view/#extensions-to-the-window-interface),
which were generally designed around the use of a single screen:
* `Element.requestFullscreen()` only supports fullscreen on the current screen
* `Window.open()` and `moveTo()/moveBy()` often clamp to the current screen
* `Window.screen` only provides information about the Window's current screen
* The `Web-exposed screen area` does not account for multiple available screens

So, web application users with multiple screens have no choice but to manually
drag windows to the appropriate screen, exit and re-enter fullscreen to use the
intended screen, and work around other limitations of web applications that can
only make use of an artificially limited visual workspace.

Also, prospective web application developers may see this as another reason to
seek [proprietary APIs](https://developer.chrome.com/apps/system_display),
[platform extensions](https://www.electronjs.org/docs/api/screen), or another
application development platform altogether.

As multi-screen devices and applications become a more common and critical part
of user experiences, it becomes more important to give developers information
and tools to leverage that expanded visual environment. This document describes
some possible incremental solutions enabling web application developers to make
use of multi-screen devices, in order to facilitate discussion and seek
concensus on a path forward.

## Use Cases

The aim of this proposal is enable better experiences for web application users
with multiple screens. Here are some use cases that inform the goals below:
* Slideshow app presents on a projector, shows speaker notes on a laptop screen
* Financial app opens a dashboard of windows across multiple monitors
* Medical app opens images (e.g. x-rays) on a high-resolution grayscale display
* Creativity app shows secondary windows (e.g. pallete) on a separate screen
* Conference room app shows controls on a touch screen device and video on a TV
* Multi-screen layouts in gaming, signage, artistic, and other types of apps
* Site optimizes content and layout when a window spans multiple screens

## Goals

The following specific goals and proposed solutions make incremental extensions
of existing window placement APIs to support extended multi-screen environments.
* Support requests to show elements fullscreen on a specific screen
  * Extend `Element.requestFullscreen()` for specific screen requests
* Support requests to place web app windows on a specific screen
  * Extend `Window.open()` and `moveTo()/moveBy()` for cross-screen coordinates
* Provide requisite information to achieve the goals above
  * Add `Screen.isExtended` to expose the presence of extended screen areas
  * Add `Screen.change`, an event fired when Screen attributes change
  * Add `Window.getScreens()` to request additional permission-gated screen info
  * Add `Screens` and `ScreenAdvanced` interfaces for additional screen info
  * Standardize common `Screen.availLeft` and `Screen.availTop` attributes
  * Add Permission API support for a new `window-placement` entry

These allow web applications to make window placement requests optimized for the
specific use case, characteristics of available screens, and user preferences.

See explorations of alternative and supplemental proposals in
[additional_explorations.md](https://github.com/webscreens/window-placement/blob/master/additional_explorations.md).

### Non-goals

* Expose extraneous or especially identifying screen information (e.g. EDID)
* Expose APIs to control the configuration of screen devices
* Place windows outside a user's view or across virtual workspaces/desktops
* Offer declarative window arrangements managed by the browser
* Open or move windows on remote displays connected to other devices
  * See the [Presentation](https://www.w3.org/TR/presentation-api/) and
    [Remote Playback](https://www.w3.org/TR/remote-playback/) APIs
* Capturing links in specific existing/new windows, etc.
  * See [Service Worker Launch Event](https://github.com/WICG/sw-launch) and
    [PWAs as URL Handlers](https://github.com/WICG/pwa-url-handler/blob/master/explainer.md)

## Support requests to show elements fullscreen on a specific screen

One of the primary use cases for Window Placement is showing fullscreen content
on the optimal screen for the given use case. An example of this is presenting a
slideshow or other media on an external display when the web application window
controlling the presentation is on the 'internal' display built into a laptop.

Our proposed solution is to support requests for a specific screen through the
[`fullscreenOptions`](https://fullscreen.spec.whatwg.org/#dictdef-fullscreenoptions)
dictionary parameter of
[`Element.requestFullscreen()`](https://fullscreen.spec.whatwg.org/#dom-element-requestfullscreen).

```webidl
dictionary FullscreenOptions {
  FullscreenNavigationUI navigationUI = "auto";

  // NEW: An optional way to request a specific screen for element fullscreen.
  ScreenAdvanced screen;
};
```

In the slideshow presentation example, the presenter could now click a button to
"Present on external display", which requests fullscreen on the intended screen,
instead of first dragging the window to that intended screen and then clicking
a button that can only request fullscreen on the current display.

```js
// Show the slideshow element fullscreen on the optimal screen.
// See the definition of getScreenForSlideshow() later in this document.
// NEW: `screen` on `fullscreenOptions` for `requestFullscreen()`.
slideshowElement.requestFullscreen({ screen: await getScreenForSlideshow() });
```

As the `screen` dictionary member is not `required`, it is implicitly optional.
Callers can omit the member altogether to use the window's current screen, which
ensures backwards compatibility with existing usage. Callers can also explicitly
pass `undefined` for the same result. Passing `null` for this optional member is
not supported and will yield a TypeError, as is typical of modern web APIs.

## Support requests to place web app windows on a specific screen

Web applications also have compelling use cases for placing non-fullscreen
content windows on screens other than the current screen. For example, a media
editing application may wish to place companion windows on a separate screen,
when the main editing window is maximized on a particular screen. The app may
determine an initial multi-screen placement based on available screen info and
application settings, or it may be restoring a user's saved window arrangement.

Existing `Window` placement APIs have varied histories and awkward shapes, but
generally offer a sufficient surface for cross-screen window placement:
* [`open()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/open)
  accepts `left`, `top`, `width`, and `height` in the
  [features](https://developer.mozilla.org/en-US/docs/Web/API/Window/open#Window_features)
  argument string
* [`moveTo(x, y)`](https://developer.mozilla.org/en-US/docs/Web/API/Window/moveTo)
  and
  [`resizeTo(width, height)`](https://developer.mozilla.org/en-US/docs/Web/API/Window/resizeTo)
  take similar values to place existing windows
* [`screenX`](https://drafts.csswg.org/cssom-view/#dom-window-screenx),
  [`screenY`](https://drafts.csswg.org/cssom-view/#dom-window-screeny),
  [`outerWidth`](https://drafts.csswg.org/cssom-view/#dom-window-outerwidth),
  and
  [`outerHeight`](https://drafts.csswg.org/cssom-view/#dom-window-outerheight)
  provide info about the current window placement

```webidl
partial interface Window {
  // NEW: `features` support `left` and `top` coordinates on other screens.
  Window? open(optional USVString url="", optional DOMString target = "_blank",
               optional DOMString features = "");
  // NEW: `x` and `y` support coordinates placing windows on other screens.
  void moveTo(long x, long y);
  // NEW: `x` and `y` support deltas placing windows on other screens.
  void moveBy(long x, long y);

  // NEW: Coordinates are defined relative to the multi-screen origin.
  readonly attribute long screenX;
  readonly attribute long screenLeft;
  readonly attribute long screenY;
  readonly attribute long screenTop;
};
```

The least invasive way to support multi-screen window placement is to specify
positional coordinates relative to a multi-screen origin (e.g. top-left of the
primary screen) in existing API surfaces. Coordinates are currently specified as
[CSS pixels](https://drafts.csswg.org/css-values-4/#px) relative to the
[web exposed screen area](https://drafts.csswg.org/cssom-view/#web-exposed-screen-area),
which refers to a singular output device. Those definitions would be updated to
clarify the behavior in multi-screen environments.

Most browser implementations already use coordinates relative to a multi-screen
origin, but may clamp placement within the current screen, rather than allowing
placement on other screens. Implementation-specific behaviors may be acceptable,
but aforementioned existing specifications would permit placing windows on other
screens, in accordance with the requested coordinates and any applicable user
permissions.

This aspect of the proposal matches the existing behavior of some browsers,
requires no API shape changes, and seems like the simplest option available, but
it has some challenges. See alternatives, considerations, and more examples in
[additional_explorations.md](https://github.com/webscreens/window-placement/blob/master/additional_explorations.md).

In the media editing application example, the user might save and close their
current project, later selecting an option to open that saved project, which
restores the multi-screen window placement configuration for that project. A
similar pattern would be useful to financial dashboards and other applications.

```js
// Save the open windows with placements when the user saves the project.
function saveOpenWindows(project, openWindows) {
  await saveWindowInIndexDB(project, openWindows.map(w => {
    return { url: w.location.href, name: w.name,
             left: w.screenX, top: w.screenY,
             width: w.outerWidth, height: w.outerHeight };
  }));
}

// Restore saved windows with placements when the user opens the project.
function restoreSavedWindows(project, openWindows) {
  for (let w of await getSavedWindowsFromIndexDB(project)) {
    let openWindow = openWindows.find(o => o.name === w.name);
    if (openWindow) {
      // NEW: `x` and `y` may be outside the window's current screen.
      openWindow.moveTo(w.left, w.top);
      openWindow.resizeTo(w.width, w.height);
    } else {
      // NEW: `left` and `top` may be outside the opener's current screen.
      window.open(w.url, w.name, `left=${w.left},top=${w.top}` +
                                 `width=${w.width},height=${w.height}`);
    }
  }
}
```

## Provide requisite information to achieve the goals above

### Add `Screen.isExtended` to expose the presence of extended screen areas

The most basic question developers may ask to support multi-screen devices is:
"Does this device have multiple screens that may be used for window placement?"
The proposed shape for this particularly valuable limited-information query is
a single `Screen.isExtended` boolean, exposed to secure contexts without an
explicit permission prompt.

```webidl
partial interface Screen {
  // NEW: Yields whether the visual workspace extends over 2+ screens.
  [SecureContext] readonly attribute boolean isExtended;
};
```

This single bit provides high value to users and developers alike, allowing
sites to offer multi-screen functionality only when it is applicable, and to
avoid requesting additional information and capabilities that are not applicable
to single-screen environments. The value of this information seems to outweigh
the risks posed by minimally increasing the device fingerprinting surface. See
more thorough Privacy & Security considerations later in this document.

In the slideshow example, the site may now offer specific UI entrypoints for
single-screen and multi-screen environments.

```js
function updateSlideshowButtons() {
  // NEW: Yields whether the visual workspace extends over 2+ screens.
  const multiScreenUI = window.screen.isExtended;  // Offer multi-screen UI?
  document.getElementById("multi-screen-slideshow").hidden = !multiScreenUI;
  document.getElementById("single-screen-slideshow").hidden = multiScreenUI;
}
```

### Add `Screen.change`, an event fired when Screen attributes change

Sites must currently poll the existing `Screen` interface for changes. This is
problematic, as polling frequently consumes battery life, polling infrequently
delays UI updates (detracting from the user experience), and the overall pattern
yields inconsistencies between sites and adds a development burden. This can
easily be solved by adding an event that is fired when screen attributes change.
The proposed shape is a `Screen.change` event, exposed to secure contexts
without an explicit permission prompt.

```webidl
// NEW: Screen inherits EventTarget.
interface Screen : EventTarget {
  // <existing Screen spec omitted for brevity>

  // NEW: An event fired when attributes of a specific screen change.
  [SecureContext] attribute EventHandler onchange;
};
```

This is useful for updating multi-screen UI entrypoints or prompting users for
additional multi-screen information when extended screens become available. It
may also be useful for adapting content or window placements to screen changes.
See Privacy & Security considerations for these events later in this document.

The slideshow example's multi-screen UI can now be updated in a change handler:

```js
let cachedScreenIsExtended = window.screen.isExtended;
window.screen.addEventListener('change', function() {
  if (cachedScreenIsExtended != window.screen.isExtended)
    updateSlideshowButtons();  // Defined in a prior explainer section.
});
```

### Add `Window.getScreens()` to request additional permission-gated screen info

Sites require information about the available screens in order to make optimal
application-specific use of that space, to save and restore the user's window
placement preferences for specific screens, or to offer users customized UI for
choosing appropriate window placements. The proposed shape of this query is a
`Window.getScreens()` method, alongside the existing `screen` attribute.

```webidl
partial interface Window {
  // NEW: Requests permission-gated access to additional screen information.
  [SecureContext] Promise<Screens> getScreens();
};
```

This method gives the web platform a surface to optionally expose an appropriate
amount of multi-screen information to web applications. By returning a promise,
user agents can asynchronously determine what amount of information to expose,
prompt users to decide, obtain underlying information lazily, and reject or
resolve accordingly.

The slideshow example can now request access to multi-screen information:

```js
// Request information for multi-screen slideshow functionality, if available.
async function startSlideshow() {
  // NEW: Feature-detect availability of additional screen information.
  if ("getScreens" in window) {
    try {
      // NEW: Requests permission-gated access to additional screen information.
      let screensInterface = await window.getScreens();
      startMultiScreenSlideshow(screensInterface);
      return;
    } catch (err) {
      // The request was denied or an error occurred.
      console.error(err.name, err.message);
    }
  }
  // If multi-screen info is not available, start a single-screen slideshow.
  startSingleScreenSlideshow();
};
```

### Add `Screens` and `ScreenAdvanced` interfaces for additional screen info

The `getScreens()` method grants access to a `Screens` interface on success.
That provides multi-screen information and change events, as well as additional
per-screen information via a `ScreenAdvanced` interface, which inherits from the
existing [`Screen`](https://drafts.csswg.org/cssom-view/#screen) interface. The
proposed shapes of these interfaces, exposed to secure contexts with explicit
permission, are outlined below:

```webidl
// NEW: Interface exposing multiple screens and additional information.
[SecureContext] interface Screens : EventTarget {
  // NEW: The set of available screens with additional per-screen info.
  readonly attribute FrozenArray<ScreenAdvanced> screens;

  // NEW: A reference to the current screen with additional info.
  readonly attribute ScreenAdvanced currentScreen;

  // NEW: An event fired when 'screens' or 'currentScreen' changes.
  // NOTE: Does not fire on changes to attributes of individual Screens.
  attribute EventHandler onchange;
};

// NEW: Interface inherits Screen and exposes additional information.
[SecureContext] interface ScreenAdvanced : Screen {
  // Shape matches commonly implemented Screen attributes that that are not yet
  // standardized; see https://developer.mozilla.org/en-US/docs/Web/API/Screen
  // Distances from a multi-screen origin (e.g. primary screen top left) to the:
  readonly attribute long left;       // Left edge of the screen area, e.g. 1920
  readonly attribute long top;        // Top edge of the screen area, e.g. 0
  // NOTE: readonly attribute long availLeft should be specified on Screen.
  // NOTE: readonly attribute long availTop should be specified on Screen.

  // New properties critical for many multi-screen window placement use cases:

  // If this screen is designated as the 'primary' screen by the OS (otherwise
  // it is 'secondary'). Useful for placing prominent vs peripheral windows.
  readonly attribute boolean isPrimary;  // e.g. true

  // If this screen is an 'internal' screen, built into the device, like a
  // laptop display. Useful for placing slideshows on external projectors and
  // controls or notes on internal laptop screens.
  readonly attribute boolean isInternal;  // e.g. false

  // The ratio of this screen's resolution in physical pixels to its resolution
  // in CSS pixels. Useful for placing windows on screens with optimal scaling
  // and appearances for a given application.
  readonly attribute float devicePixelRatio;  // e.g. 2

  // A temporary generated per-origin unique ID; reset when cookies are deleted.
  // Useful for persisting window placement preferences for certain screens.
  readonly attribute DOMString id;

  // The set of PointerTypes supported by the screen. Useful for placing control
  // panels on touch-screens and drawing surfaces on screens with pen support.
  readonly attribute FrozenArray<PointerType> pointerTypes;  // e.g. [ "touch" ]
};
```

These interfaces provide the most crucial information for placing windows in
extended multi-screen environments. The relative bounds establish a coordinate
system for cross-screen window placement, while newly exposed display device
properties allow applications to restore or choose window placements. See
relevant Privacy & Security considerations later in this document.

The slideshow example can now define `startMultiScreenSlideshow()` to provide an
enhanced web application experience, commonplace among non-web counterparts:

```js
// Place slides and notes windows on separate screens, if available.
async function startMultiScreenSlideshow(screensInterface) {
  // NEW: Use granted multi-screen info; cache the count for comparison later.
  let cachedScreenCount = screensInterface.screens.length;

  // NEW: Handle changes to the set of available screens or current screen:
  screensInterface.addEventListener('change', function() {
    // NEW: Check if the change was fired because a new screen was connected.
    if (screensInterface.screens.length > cachedScreenCount) {
      // Offer to move the presentation to the newly connected screen.
    }
    cachedScreenCount = screensInterface.screens.length;
  });

  if (screensInterface.screens.length == 1) {
    // If only one screen is available, start a single-screen slideshow.
    startSingleScreenSlideshow();
    return;
  }

  // Prefer an external screen or a secondary screen for the slideshow window.
  const slidesScreen =
      screensInterface.screens.find(s => !s.internal)
      ?? screensInterface.screens.find(s => !s.isPrimary);

  // Prefer an internal screen, or any other screen for the notes window.
  const otherScreens = screensInterface.screens.filter(s => s != slidesScreen);
  const notesScreen = otherScreens.find(s => s.internal) ?? otherScreens[0];

  // TODO: Define this with requestFullscreen and/or window.open/moveTo?
  placeSlidesAndNotesOnPreferredScreens(slidesScreen, notesScreen);
}
```

Similar screen selection logic is critical for other web application use cases:

```js
// Get a touch-screen for a conference room app's touch-based interface.
let touchScreen = screensInterface.screens.find(s => s.touchSupport);
```

```js
// Get a wide color gamut screen for a creativity app's color balancing window.
let wideColorGamutScreen = screensInterface.screens.reduce(
    (a, b) => a.colorDepth > b.colorDepth ? a : b);
```

```js
// Get a high-resolution screen for a medical app's image inspection window.
let highResolutionScreen = screensInterface.screens.reduce(
    (a, b) => a.width*a.height > b.width*b.height ? a : b);
```

```js
// Get screens in left-to-right order for a signage app's multi-screen layout.
let sortedScreens = screensInterface.screens.sort((a, b) => b.left - a.left);
```

NOTE: The `element.requestFullscreen()` algorithm could reasonably be updated to
support being triggered by user-generated `Screens.onchange` events, matching
[existing behavior](https://fullscreen.spec.whatwg.org/#dom-element-requestfullscreen)
when triggered by user-generated `ScreenOrientation.onchange` events. This would
allow sites to request fullscreen or change the screen used for fullscreen when
users connect a new screen.

NOTE: Screen mirroring relationships are not currently exposed; Screens may omit
destinations and only expose source screens of screen mirroring relationships.

### Standardize common `Screen.availLeft` and `Screen.availTop` attributes

The [`Screen`](https://drafts.csswg.org/cssom-view/#the-screen-interface)
interface currently specifies the sizes of the
[`web-exposed screen area`](https://drafts.csswg.org/cssom-view/#web-exposed-screen-area)
and the
[`web-exposed available screen area`](https://drafts.csswg.org/cssom-view/#web-exposed-available-screen-area)
(which excludes screen areas reserved for system UI, like toolbars or taskbars).
Unfortunately, the available area's bounds cannot be determined, since its
origin is unspecified and assuming zeroes is incorrect.

Even in single-screen environments, the top or left edge of the screen may be
reserved for system UI, so `window.moveTo(0, 0)` may specify a window position
outside the available area, and undesirably trigger user-agent-defined clamping.
Sites should be able to determine the screen's available area, to accurately
formulate and better anticipate outcomes of standardized placement requests.

To solve this, most browsers also expose two unstandardized Screen attributes,
[`availLeft`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/availLeft)
and
[`availTop`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/availTop),
which are critical for discerning the region available for placing windows.
These attributes should be standardized on the `Screen` interface itself to
support valid single-screen (or same-screen) placement requests, to support
existing scripts that already use these unstandardized attributes, and to
support new multi-screen window placement requests.

```webidl
partial interface Screen {
  // NEW: Shape matches commonly implemented Screen attributes that are not yet
  // standardized; see https://developer.mozilla.org/en-US/docs/Web/API/Screen
  // Distances from a multi-screen origin (e.g. primary screen top left) to the:
  readonly attribute long availLeft;  // Left edge of the available screen area, e.g. 1920
  readonly attribute long availTop;   // Top edge of the available screen area, e.g. 0
};
```

This allows sites to better determine the `web-exposed available screen area`,
and formulate more coherent and predictable window placement requests.

```js
// NEW: Move the window to a position known to be in available screen space.
window.moveTo(screen.availLeft + offsetX, screen.availTop + offsetY);
```

See prior discussion regarding coordinates in the section titled
"Support requests to place web app windows on a specific screen".

### Add Permission API support for a new `window-placement` entry

Sites may wish to know whether users have already granted or denied a requisite
permission before attempting to access gated information and capabilities. The
proposed shape is adding a `PermissionName` entry and corresponding support via
the `query()` method of the [Permission API](https://w3c.github.io/permissions).

```webidl
enum PermissionName {
  // ...
  "window-placement",
  // ...
};
```

This allows sites to educate users that haven't been prompted, provide seamless
cross-screen support for users that have already granted permission, and respect
users that have already denied the permission.

```js
navigator.permissions.query({name:'window-placement'}).then(function(status) {
  if (status.state === "prompt")
    showMultiScreenEducationalUI();
  else if (status.state === "granted")
    showMultiScreenUI();
  else  // status.state === "denied"
    showSingleScreenUI();
});
```

### Open questions

* Would changes to the existing synchronous methods break critical assumptions?
  * Do any sites expect open/move coordinates to be local to the current screen?
* Is there value in supporting windows placements spanning multiple screens?
  * Suggest normative behavior for choosing a target display and clamping?
* How should `ScreenAdvanced` references behave when screens are disconnected?
  * Does checking `cachedReference in screensInterface.screens` suffice?
  * Should there be a `ScreenAdvanced.isConnected`?
* How can objects in the screens array be consistently ordered?
  * Is sorting by (`left`, `top`) sufficient, even for mirrored screens?

## Privacy & Security

This proposal exposes new information about the screens connected to a device,
increasing the [fingerprinting](https://w3c.github.io/fingerprinting-guidance)
surface of users, especially those with multiple screens consistently connected
to their devices. As one mitigation of this privacy concern, the exposed screen
properties are limited to the minimum needed for common placement use cases.
Further, new information is limited to secure contexts.

New window placement capabilities themselves may pose additional privacy and
security considerations; for example, showing sensitive content on unexpected
screens, hiding unwanted windows on less conspicuous screens, or otherwise using
cross-screen placements to act in deceptive, abusive, or annoying manners.

To help mitigate these concerns, user permission should be required for sites to
access nontrivial multi-screen information and place windows on other screens.
Given the API shape proposed above, user agents could reasonably prompt users
when sites call `getScreens()`, fulfilling the promise if the user accepts the
prompt, and rejecting the promise if the user denies access. If the permission
is not already granted, cross-screen placement requests could fall back to
same-screen placements, matching pre-existing behavior of some user agents. The
amount of information exposed to a given site would be at the discretion of
users and their agents.

The `Screen.isExtended` boolean is exposed without explicit permission checks,
as this minimal single bit of information supports some critical features for
which a permission prompt would be obtrusive (e.g. show/hide multi-screen UI
entry points like “Show on another screen”), and helps avoid unnecessarily
prompting single-screen users for inapplicable information and capabilities.
This generally follows a TAG design principle for
[device enumeration](https://w3ctag.github.io/design-principles/#device-enumeration):

>When designing API which allows users to select a device, it may be necessary
to also expose the fact that there are devices are available to be picked. This
does expose one bit of fingerprinting data about the user’s environment to
websites, so it isn’t quite as safe as an API which doesn’t have such a feature.

>The trade-off is that by allowing websites this extra bit of information, the
API lets authors make their user interface less confusing. They can choose to
show a button to trigger the picker only if at least one device is available.

A point to note here is that many user agents already effectively expose the
presence of multiple screens to windows located on secondary screens, where
`window.screen.availLeft|Top` >> 0. Script access of this bit is a detectable
[active fingerprinting](https://w3c.github.io/fingerprinting-guidance/#active)
signal, which may be observed and blocked by the user agent.

The new `onchange` events pose a slight risk by making
[ephemeral fingerprinting](https://github.com/asankah/ephemeral-fingerprinting)
easier, but scripts can already achieve the same result by polling for changes
to `window.screen`. This risk could be partially mitigated by delaying event
dispatch for hidden documents until such documents are no longer hidden.

User agents can generally measure and otherwise intervene when sites request the
newly proposed information or use the newly proposed capabilities.

Alternative API shapes giving less power to sites were considered, but offer
poor experiences for users and developers (e.g. prompting users to pick a
screen, requiring declarative screen rankings from developers). Few, if any,
alternatives exist for non-fullscreen placement of app windows on any connected
screen. The proposed API shape seems like the most natural extension of existing
APIs to support a more complete screen environment, and requires a reasonable
permission. Future work may include ways to query for more limited multi-screen
information to let sites voluntarily minimize their information exposure.

Some other notes:
- A user gesture is typically already required for `Element.requestFullscreen()`
  and `Window.open()`, this just adds permission-gated multi-screen support.
- Existing subframe capabilities (e.g. element fullscreen with ‘allowfullscreen’
  feature policy, and window.open) should support cross-screen info and
  placements with permission.
- Gating pre-existing information exposure and placement capabilities on the
  newly proposed permission may be reasonable.
- Placement on a different screen from the active window is less likely to
  create additional clickjacking risk for users, since the user's cursor or
  finger is likely to be co-located with the current screen and window, not on
  the separate target screen.
- ScreenAdvanced IDs generally follow patterns of other device information APIs.
- A new affordance for fullscreen requests on `Screens.onchange` events follows
  the precedent of `ScreenOrientation.onchange`, which is not permission gated.

See
[security_and_privacy.md](https://github.com/webscreens/window-placement/blob/master/security_and_privacy.md)
for additional explorations of privacy and security concerns.

## Related explainers:
| Name | Link |
|------|------|
| Window Segments Enumeration API | [Explainer](https://github.com/webscreens/window-segments/blob/master/EXPLAINER.md)|
| Screen Fold API | [Explainer](https://github.com/SamsungInternet/Explainers/blob/master/Foldables/FoldState.md), [Unofficial Draft Spec](https://w3c.github.io/screen-fold/) |
| Visual Viewport API | [Draft Community Group Report](https://wicg.github.io/visual-viewport/), [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Visual_Viewport_API) |