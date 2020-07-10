# Multi-Screen Window Placement on the Web

## Introduction

Operating systems generally allow users to connect multiple screens to a single
device and arrange them virtually to extend the overall visual workspace. 

A variety of applications use platform tools to place windows in multi-screen
environments, but web application developers are limited by existing APIs, like
[`Screen`](https://drafts.csswg.org/cssom-view/#the-screen-interface) and
[`Window`](https://drafts.csswg.org/cssom-view/#extensions-to-the-window-interface),
which were generally designed around the use of a single screen.

TODO: Link to specs, or flesh out existing limitations in more detail elsewhere?
* `Element.requestFullscreen()` only supports fullscreen on the current screen
* `Window.open()` and `moveTo()/moveBy()` generally clamp to the current screen
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

The aim of this project is enable better experiences for web application users
with multiple screens. Some use cases that inform this proposal are:
* Slideshow app presents on a projector, shows speaker notes on a laptop screen
* Financial app opens a dashboard of windows across multiple monitors
* Medical app open images (e.g. x-rays) on a high-resolution grayscale display
* Creativity app shows secondary windows (e.g. pallete) on secondary screens
* Conference room app shows controls on a touch screen device and video on a TV
* Site optimizes content and layout when a window spans multiple screens

## Goals

The following specific goals and proposed solutions make incremental extensions
of existing window placement APIs to support expanded multi-screen environments. 
* Support requests to show element fullscreen on any screen
  * Extend `Element.requestFullscreen()` for specific screen requests
* Support requests to place web app windows on any screen
  * Extend `Window.open()` and `moveTo()/moveBy()` for cross-screen coordinates
* Provide requisite info about available screens to achieve the goals above
  * Add `isMultiScreen()` to expose whether a device has multiple screens
  * Add `getScreens()` to expose information about available screens
  * Add a `screenschange` event, fired on screen connection or property changes

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

## Support requests to show element fullscreen on any screen

One of the primary use cases for Window Placement is showing fullscreen content
on the optimal screen for the given use case. An example of this is presenting a
slideshow or other media on an external display when the web application window
controlling the presentation is on the internal display of a laptop.

Our proposed solution is to support requests for a specific screen through the
[`fullscreenOptions`](https://fullscreen.spec.whatwg.org/#dictdef-fullscreenoptions)
dictionary parameter of
[`Element.requestFullscreen()`](https://fullscreen.spec.whatwg.org/#dom-element-requestfullscreen).

```js
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
// NEW: `screen` on `fullscreenOptions` for `requestFullscreen()`.
slideshowElement.requestFullscreen({ screen: getScreenForSlideshow() });
```

As the `screen` dictionary member is not `required`, it is implicitly optional.
Callers can omit the member altogether to use the window's current screen, which
ensures backwards compatibility with existing usage. Callers can also explicitly
pass `undefined` for the same result. Passing `null` for this optional member is
not supported and will yield a TypeError, as is typical of modern web APIs.

## Support requests to place web app windows on any screen

Web applications also have compelling use cases for placing non-fullscreen
content windows on screens other than the current screen. For example, a media
editing application may wish to place companion windows on a secondary screen,
when the main editing window is maximized on the primary screen. The app may
determine an initial multi-screen placement based on available screen info and
application settings, or it may be restoring a user's saved window arrangement.

Existing [`Window`][2] placement APIs have varied histories and awkward shapes,
but generally offer a sufficient surface for cross-screen window placement:
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
  [`outerHeight`](https://drafts.csswg.org/cssom-view/#dom-window-outerwidth)
  provide info about the current window placement

The least invasive way to support multi-screen window placement is to specify
positional coordinates relative to the origin of the primary screen within the
existing API surfaces. Coordinates are currently specified as 
[CSS pixels](https://drafts.csswg.org/css-values-4/#px) relative to the
[web exposed screen area](https://drafts.csswg.org/cssom-view/#web-exposed-screen-area),
which refers to a singular output device. Those definitions should be updated to
clarify the behavior in multi-screen environments.

Most browser implementations already use coordinates relative to the primary
screen, but may clamp placements to the current screen, rather than allowing
placement on other screens. Implementation-specific behaviors may be acceptable,
but specifications should permit placing windows on other screens, in accordance
with the requested coordinates and any applicable user permissions.

This aspect of the proposal matches the existing behavior of some browsers,
requires no API shape changes, and seems like the simplest option available, but
it has some challenges. See alternatives, considerations, and more examples in
[additional_explorations.md](https://github.com/webscreens/window-placement/blob/master/additional_explorations.md).

In the media editing application example, the user might save and close their
current project, later selecting an option to open that saved project, which
restores the multi-screen window placement configuration for that project.

```js
// Save the open window with placements when the user saves the project.
function saveOpenWindows(project, openWindows) {
  for (w of openWindows) {
    saveWindowInIndexDB(project,
                        { url: w.location.href, name: w.name,
                          left: w.screenX, top: w.screenY,
                          width: w.outerWidth, height: w.outerHeight });
  }
}

// Restore saved windows with placements when the user opens the project.
function restoreSavedWindows(project, openWindows) {
  for (w of getSavedWindowsFromIndexDB(project)) {
    let openWindow = openWindows.find((o)=>{return o.name === w.name;});
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

TODO: Financial dashboard app, presentation app examples?

TODO: Use example code here that directly employs proposals defined below?

```js
// NEW: `getScreens()` provides requisite info; see the proposal below.
const touchScreen = (await getScreens()).find((s)=>{return s.touchSupport;});
// Open a window on a touch-compatible display, if one exists.
// NEW: `left` and `top` may be outside the window's current screen.
window.open(url, ``, `left=${touchScreen.availLeft},top=${touchScreen.availTop}`);
```

```js
// NEW: `getScreens()` provides requisite info; see the proposal below.
const otherScreen = (await getScreens()).find((s)=>{
    return s.availLeft != window.screen.availLeft ||
           s.availTop != window.screen.availTop;});
// Move the window to another screen (e.g. user clicked "swap screens").
// NEW: `x` and `y` may be outside the window's current screen.
window.moveTo(otherScreen.availLeft + window.screenLeft,
              otherScreen.availTop + window.screenTop);
```

## Provide requisite info about available screens to achieve the goals above

### Add `isMultiScreen()` to expose whether a device has multiple screens

The most basic question developers may ask to support multi-screen devices is:
"Does this device have multiple screens that may be used for window placement?"
The proposed shape for this particularly valuable limited-information query is
a `Window.isMultiScreen()` method, alongside the existing `screen` attribute.

```js
partial interface Window {
  // NEW: Returns whether the device has multiple connected screens on success. 
  Promise<boolean> isMultiScreen();  // UAs may prompt for permission.
};
```

This method exposes the minimum information needed to engage multi-screen users,
and to avoid requesting information and capabilities that are not applicable to
single-screen users. In the slideshow example, the site may offer specific UI
entrypoints for single-screen and multi-screen users.

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

```js
partial interface Window {
  // NEW: Returns a snapshot of information about connected screens on success.
  Promise<sequence<ScreenInfo>> getScreens();  // UAs may prompt for permission.
};
```

The promise resolves to an array of screen information dictionaries on success,
and rejects otherwise (e.g. if permission is denied). `ScreenInfo` dictionaries
are static snapshots of screen configuration information, shaped similar to the
existing [`Screen`](https://drafts.csswg.org/cssom-view/#screen) interface, with
additional properties that provide requisite information for many window
placement use cases.

```js
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
  // Useful for understanding the relative screen layout for window placements.
  // Distances from the origin (top left corner) of the primary screen to the: 
  long left;       // Left edge of the screen area, e.g. 1920
  long top;        // Top edge of the screen area, e.g. 0
  long availLeft;  // Left edge of the available screen area, e.g. 1920
  long availTop;   // Top edge of the available screen area, e.g. 0

  // New properties critical for many multi-screen window placement use cases.
  boolean primary;       // If this screen is the primary screen, e.g. true
                         // Useful for placing prominent vs peripheral windows.
  boolean internal;      // If this screen is internal (built-in), e.g. false
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
                         // Useful for placing control windows on touch-screens.
};
```

`ScreenInfo` objects retrieved from `getScreens()` are integral for multi-screen
window placement use cases. The relative bounds establish a coordinate system
for cross-screen window placement, while the newly exposed properties of the
display devices allow applications to restore or choose window placements.

In the slideshow example, the site could use multi-screen information to place
presenter and audience windows optimally with a single click, which is a common
feature of existing non-web slideshow applications.

```js
document.getElementById("multi-screen-slideshow").onclick = async function() {
  // NEW: Returns a snapshot of information about connected screens on success.
  const screens = await window.getScreens();
  // Place slides on an external screen, or failing that, any secondary screen.
  const s1 = screens.find(s => !s.internal) ?? screens.find(s => !s.primary);
  // Place notes on an internal screen, or failing that, any screen besides s1.
  const s2 = screens.find(s => s.internal) ?? screens.find(s => s != s1);
  // TODO: Demonstrate more complex screen selection logic, e.g. favor external
  // touch-screens if there is no internal screen for notes, favor external
  // screens with resolution and scaling most suitable for a slideshow, etc.
  // TODO: Show requestFullscreen or window.open/moveTo usage here?
  if (s1 && s2 && s1 != s2) {
    placeSlidesOn(s1);  // TODO: Define this (in another section?)
    placeNotesOn(s2);   // TODO: Define this (in another section?)
  } else {
    singleScreenSlideshow();  // TODO: Define this (in another section?)
  }
```

### Add a `screenschange` event, fired on screen connection or property changes

Since `getScreens()` returns a static snapshot, sites need an event, fired when
the set of screens or their properties change, to avoid polling for changes. The
proposed shape is a `screenschange` event on `Window`, alongside the existing
`screen` attribute.

```js 
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
  updateSlideshowButtons();  // Defined in a prior explainer section.

  // TODO: Define this hand-waving example code.
  if (inSlideShow() && screenShowingSlides() != bestScreenForSlides())
    moveSlideshowToBestScreen();
});
```

### Open questions

* Would changes to the existing synchronous methods break critical assumptions?
  * Do any sites expect open/move coordinates to be local to the current screen?
* Is there value in supporting windows placements spanning multiple screens?
  * Suggest normative behavior for choosing a target display and clamping?
* Add an id to the Screen interface for comparison with ScreenInfo dictionaries?

## Privacy & Security

This API exposes new device infomation to the web and adds new web platform
capabilites that require non-trivial privacy and security considerations. See
[security_and_privacy.md](https://github.com/webscreens/window-placement/blob/master/security_and_privacy.md)
for detailed explorations of specific privacy and security concerns.
