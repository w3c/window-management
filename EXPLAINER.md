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
of existing window placement APIs to support expanded multi-screen environments. 
* Support requests to show elements fullscreen on a specific screen
  * Extend `Element.requestFullscreen()` for specific screen requests
* Support requests to place web app windows on a specific screen
  * Extend `Window.open()` and `moveTo()/moveBy()` for cross-screen coordinates
* Provide requisite information to achieve the goals above
  * Add `isMultiScreen()` to expose whether a device has multiple screens
  * Add `getScreens()` to expose information about available screens
  * Add a `screenschange` event, fired on screen connection or property changes
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
  ScreenInfo screen;
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
editing application may wish to place companion windows on a speparate screen,
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
similar pattern would be useful to financial dashboards and other appplications.

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

### Add `isMultiScreen()` to expose whether a device has multiple screens

The most basic question developers may ask to support multi-screen devices is:
"Does this device have multiple screens that may be used for window placement?"
The proposed shape for this particularly valuable limited-information query is
a `Window.isMultiScreen()` method, alongside the existing `screen` attribute.

```webidl
partial interface Window {
  // NEW: Returns whether the device has multiple connected screens on success. 
  Promise<boolean> isMultiScreen();  // UAs may prompt for permission.
};
```

This method exposes the minimum information needed to engage multi-screen users,
and to avoid requesting information and capabilities that are not applicable to
single-screen users. By returning a promise, user agents can asynchronously
determine whether sites may access this information, prompt users to decide,
calculate the resulting value lazily, and reject or resolve accordingly.

In the slideshow example, the site may offer specific UI entrypoints for
single-screen and multi-screen users.

```js
async function updateSlideshowButtons() {
  // NEW: Returns whether the device has multiple connected screens on success. 
  const multiScreenUI = await window.isMultiScreen();  // Show multi-screen UI?
  document.getElementById("multi-screen-slideshow").hidden = !multiScreenUI;
  document.getElementById("single-screen-slideshow").hidden = multiScreenUI;    
}
```

### Add `getScreens()` to expose information about available screens

Sites require information about the available screens in order to make optimal
application-specific use of that space, to save and restore the user's window
placement preferences for specific screens, or to offer users customized UI for
choosing appropriate window placements. The proposed shape of this query is a
`Window.getScreens()` method, alongside the existing `screen` attribute.

```webidl
partial interface Window {
  // NEW: Returns a snapshot of information about connected screens on success.
  Promise<sequence<ScreenInfo>> getScreens();  // UAs may prompt for permission.
};
```

`ScreenInfo` dictionaries are static snapshots of screen configuration
information, shaped similar to the existing
[`Screen`](https://drafts.csswg.org/cssom-view/#screen) interface, with
additional properties that can optionally provide requisite information for many
window placement use cases.

```webidl
dictionary ScreenInfo {
  // Shape matches https://drafts.csswg.org/cssom-view/#the-screen-interface
  long availWidth;           // Width of the available screen area, e.g. 1920
  long availHeight;          // Height of the available screen area, e.g. 1032
  long width;                // Width of the screen area, e.g. 1920
  long height;               // Height of the screen area, e.g. 1080
  unsigned long colorDepth;  // Bits allocated to colors for a pixel, e.g. 24
  unsigned long pixelDepth;  // Bits allocated to colors for a pixel, e.g. 24

  // Shape roughly matches https://w3c.github.io/screen-orientation
  OrientationType orientationType;  // Orientation type, e.g. "portrait-primary"
  unsigned short orientationAngle;  // Orientation angle, e.g. 0

  // Shape matches https://developer.mozilla.org/en-US/docs/Web/API/Screen
  // Critical for understanding relative screen layouts for window placement.
  // Distances from a multi-screen origin (e.g. primary screen top left) to the: 
  long left;       // Left edge of the screen area, e.g. 1920
  long top;        // Top edge of the screen area, e.g. 0
  long availLeft;  // Left edge of the available screen area, e.g. 1920
  long availTop;   // Top edge of the available screen area, e.g. 0

  // New properties critical for many multi-screen window placement use cases.
  boolean primary;       // If this screen is designated as the 'primary' screen
                         // by the OS (otherwise it is 'secondary'), e.g. true
                         // Useful for placing prominent vs peripheral windows.
  boolean internal;      // If this screen is an 'internal' display, built into
                         // the device, like a laptop screen, e.g. false
                         // Useful for placing slideshows on external projectors
                         // and controls/notes on internal laptop screens.
  float scaleFactor;     // Ratio between physical pixels and device
                         // independent pixels for this screen, e.g. 2
                         // Useful for placing windows on screens with optimal
                         // scaling and appearances for a given application.
  DOMString id;          // A temporary, generated per-origin unique ID; resets
                         // when cookies are deleted. Useful for persisting user
                         // window placements preferences for certain screens.
  boolean touchSupport;  // If the screen supports touch input, e.g. false
                         // Useful for placing control panels on touch-screens.
};
```

This method gives the web platform a surface to optionally expose an appropriate
amount of multi-screen information to web applications. By returning a promise,
user agents can asynchronously determine what amount of information to expose,
prompt users to decide, calculate the resulting values lazily, and reject or
resolve accordingly.

`ScreenInfo` objects retrieved from `getScreens()` are integral for multi-screen
window placement use cases. The relative bounds establish a coordinate system
for cross-screen window placement, while the newly exposed properties of the
display devices allow applications to restore or choose window placements.

This API can be used to define `getScreenForSlideshow()`, referenced in an
earlier example. More advanced slideshow web applications could place slides and
notes windows on separate preferred screens, like existing non-web counterparts.

```js
// Get the preferred screen for showing a fullscreen slideshow presentation.
async function getScreenForSlideshow() {
  // NEW: Returns a snapshot of information about connected screens on success.
  let screens = await window.getScreens();
  // Prefer an external screen, or failing that, a secondary screen.
  return screens.find(s => !s.internal) ?? screens.find(s => !s.primary);
}

document.getElementById("multi-screen-slideshow").onclick = async function() {
  const s1 = getScreenForSlideshow();
  // Place notes on an internal screen, or failing that, any screen besides s1.
  const screens_without_s1 = (await window.getScreens()).filter(s => s != s1);
  const s2 = screens_without_s1.find(s => s.internal) ?? screens_without_s1[0];
  // TODO: Demonstrate more complex screen selection logic, e.g. favor external
  // touch-screens if there is no internal screen for notes, favor external
  // screens with resolution and scaling most suitable for a slideshow, etc.
  // TODO: Define this with requestFullscreen and/or window.open/moveTo?
  placeSlidesAndNotesOnPreferredScreens(s1, s2);
}
```

Similar screen selection logic is critical for other web applications use cases:

```js
// Get a touch-screen for a conference room app's touch-based interface.
let touchScreen = (await window.getScreens()).find(s => s.touchSupport);
```

```js
// Get a wide color gamut screen for a creativity app's color balancing window.
let wideColorGamutScreen = (await window.getScreens()).reduce(
    (a, b) => a.colorDepth > b.colorDepth ? a : b);
```

```js
// Get a high-resolution screen for a medical app's image inspection window.
let highResolutionScreen = (await window.getScreens()).reduce(
    (a, b) => a.width*a.height > b.width*b.height ? a : b);
```

```js
// Get screens in left-to-right order for a signage app's multi-screen layout.
let sortedScreens = (await window.getScreens()).sort((a, b) => b.left - a.left);
```

TODO: Refine and expand upon these examples.

### Add a `screenschange` event, fired on screen connection or property changes

Since `getScreens()` returns a static snapshot, sites need an event, fired when
the set of screens or their properties change, to avoid polling for changes. The
proposed shape is a `screenschange` event on `Window`, alongside the existing
`screen` attribute.

```webidl
partial interface Window {
  // NEW: An event fired when the connected screens or their properties change.
  attribute EventHandler onscreenschange;
};
```

This is useful for updating multi-screen UI entrypoints when screens are
connected or disconnected. It may also be useful for optimizing existing window
placements to accommodate screen property changes.

```js
window.addEventListener('screenschange', async function() {
  await updateSlideshowButtons();  // Defined in a prior explainer section.

  // TODO: Define this hand-waving example code.
  if (inSlideShow() && screenShowingSlides() != bestScreenForSlides())
    moveSlideshowToBestScreen();
});
```

### Add Permission API support for a new `window-placement` entry

Sites may wish to know whether users have already granted or denied a requisite
permission before attempting to access gated information and capabilites. The
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
* Add an id to the Screen interface for comparison with ScreenInfo dictionaries?

## Privacy & Security

This proposal exposes new information about the screens connected to a device,
increasing the [fingerprinting](https://w3c.github.io/fingerprinting-guidance)
surface of users, especially those with multiple screens consistently connected
to their devices. As one mitigation of this privacy concern, the exposed screen
properties are limited to the minimum needed for common placement use cases.

New window placement capabilities themselves may pose additional privacy and
security considerations; for example, showing sensitive content on unexpected
screens, hiding unwanted windows on less conspicuous screens, or otherwise using
cross-screen placements to act in deceptive, abusive, or annoying manners.

To help mitigate these concerns, user permission should be required for sites to
get multi-screen information and place windows on other screens. Given the API
shape proposed above, user agents could reasonably prompt users when sites call
`getScreens()`, fulfilling the promise with requisite information for
cross-screen placement requests if the user accepts the prompt, and rejecting
the promise if the user denies access. If the permission is not already granted,
cross-screen placement requests could fall back to same-screen placements,
matching pre-existing behavior of some user agents. The amount of information
exposed to a given site would be at the discretion of users and their agents,
which may expose no new information, or subsets for more limited use cases.

The `isMultiScreen()` method could fulfill its promise without a user prompt,
exposing a minimal single bit of information to support some critical features
(e.g. show/hide multi-screen entry points like “Show on another screen”), and to
avoid unnecessarily prompting single-screen users for inapplicable information
and capabilities. Similarly, `screenschange` events that change the result of
`isMultiScreen()` queries could be fired without permission gates, to obviate
the need for sites to poll that method; and that could be limited to sites that
have previously called `isMultiScreen()`.

User agents can measure and otherwise intervene when sites request the newly
proposed information or use the newly proposed capabilities.

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
- ScreenInfo IDs generally follow patterns of other device information APIs.

See 
[security_and_privacy.md](https://github.com/webscreens/window-placement/blob/master/security_and_privacy.md)
for additional explorations of privacy and security concerns.
