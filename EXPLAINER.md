# Window Placement

## Abstract

Operating systems and their window managers almost universally offer the ability
for users to connect multiple physical displays to a single computing device and
virtually arrange them in a 2D plane, extending the overall visual workspace.

The web platform currently offers some window placement capabilities through the
[`Window`][2] interface, but it is limited in a number of ways that are
detrimental to users and developers. Even with information about the overall
screen space from the proposed [Screen Enumeration API][3], web developers still
have no reliable path for opening and moving windows across displays.

The ability to open and move windows across the full set of connected displays
is unstandardized and the current behavior is inconsistent between implementers.
Further, existing APIs offer no controls for the state (eg. maximized, restored)
of windowed web applications.

As multi-display computing and web applications become more common and critical
parts of user experiences, it becomes more important to give web developers the
information and tools to manage their content in modern windowing environments.

## Use cases

Given information about connected screens from the [Screen Enumeration API][3]:
```js
const screens = await getScreens();
```

Various applications wish to use multiple screens to show windows:
* Present slides on a projector, open speaker notes on the laptop's screen
* Launch and manage a dashboard of financial windows across multiple monitors
* Open windows to view medical images (eg. x-rays) on the appropriate screens
* Creativity apps showing pallete/preview windows on multiple screens
* Optimize content when a window spans multiple screens with varying properties

## Current goals, Possible future goals, and non-goals

### Current goals

The current goals are relatively limited in scope. They aim to provide basic
affordances for web applications to show content across the set of connected
displays, by using existing API surfaces and paradigms.
* Show an element fullscreen on any connected display
* Open windows on and move windows to any connected display

### Possible future goals

Future goals may provide additional capabilities and more ergonomic APIs for web
applications to manage windows. See explorations of tentative future goals in
[additional_explorations.md](https://github.com/webscreens/window-placement/blob/master/additional_explorations.md).
* Offer more ergonomic APIs
* Extend window state and display APIs
* Surface events on window bounds or state changes
* Support multiple fullscreen elements from a single document
* Extend user activation affordances
* Allow window placements that span two or more screens

### Non-goals

* Open or move windows out of view from the user's screens
* Open or move windows across virtual workspaces/desktops
* Offer declarative window arrangements managed by the browser
* Support explicit z-ordering, such as an `"alwaysOnTop"` window state
* Open or move windows on remote displays connected to other devices
  * See the [Presentation](https://www.w3.org/TR/presentation-api/) and
    [Remote Playback](https://www.w3.org/TR/remote-playback/) APIs
* Capturing links in specific existing/new windows, etc.
  * See [Service Worker Launch Event](https://github.com/WICG/sw-launch)
  * See [PWAs as URL Handlers](https://github.com/WICG/pwa-url-handler/blob/master/explainer.md)

## Proposals

### Show an element fullscreen on any connected display

This aspect of the proposal would add an optional Screen member to the optional
[`fullscreenOptions`](https://developer.mozilla.org/en-US/docs/Web/API/FullscreenOptions)
parameter of [`Element.requestFullscreen()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/requestFullScreen).
See the proposed IDL change and a JS usage example below:

```js
dictionary FullscreenOptions {
  FullscreenNavigationUI navigationUI = "auto";

  // https://github.com/webscreens/window-placement
  Screen? screen;
};
```

```js
// NEW: `getScreens()` provides requisite info; see the Screen Enumeration API.
const screens = await getScreens();
// Show the element fullscreen on an external display, if one exists.
// NEW: `screen` on `fullscreenOptions` for `requestFullscreen()`.
myElement.requestFullscreen({ screen: screens.find((s)=>{return !s.internal;});
```

It may be reasonable for subsequent requestFullscreen calls targetting a new
screen to move the existing fullscreen window to the new target screen.

Note: It may not be feasible or straightforward for multiple elements in the
same document to show as fullscreen windows on separate screens simultaneously.

### Open windows on and move windows to any connected display

Existing [`Window`][2] methods have complex histories and awkward shapes, but
generally already offer a sufficient surface for cross-screen window placement:
[`open()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/open) accepts
`left` and `top` in the features argument string, and
[`moveTo()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/moveTo)
accepts similar `x` and `y` parameters.

This aspect of the proposal aims to support cross-screen window coordinates
within the existing web-exposed API. No IDL changes are required, and this only
requires implementer-specific behavior changes to some browsers.

```js
// NEW: `getScreens()` provides requisite info; see the Screen Enumeration API.
const touchScreen = (await getScreens()).find((s)=>{return s.touchSupport;});
// Open a window on a touch-compatible display, if one exists.
// NEW: `left` and `top` may be outside the window's current screen.
window.open(url, ``, `left=${touchScreen.availLeft},top=${touchScreen.availTop}`);
```

### Improve existing API specifications and implementations

As eluded to above, the existing `Window` specifications do not provide reliable
mechanisms for multi-screen placement. Notably,
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
  * Specify `Window` coordinates as relative to the primary screen
  * Constrain the allowed user-agent-defined clamping for `Window` methods
* Encourage implementers to use reasonable and compatible cross-screen behavior
  * Foster broader adoption of interoperable cross-screen behaviors without
    making spec changes
  * Possibly add non-normative language to specs regarding cross-screen support

## Additional Thoughts

### Limitations

Changes allowing the more permissive placement of windows may still be subject
to limitations that could be left to the discretion of implementers.

For example, some implementations may reasonably restrict window movement to
certain types of windows, (e.g. only application 'popup' windows and not tabbed
browser windows) and only afford additional functionality under certain
circumstances (e.g. only in secure contexts, only after some user interaction,
limiting the number of interactions, etc.).

Limitations could also reasonably be imposed at the implementation level
regarding the placement of windows extending outside the bounds of a single
screen. While there may be valid use cases for placing a window such that it
occupies the full bounds of two or more screens, there's less apparent reason
and greatly more risk for allowing placement of windows partly outside the union
of `Screen` bounds. For example, implementer may still wish to clamp requested
window placement bounds within the union of `Screen` bounds, preventing the
placement of windows outside the visible screen space.

### Respecting intents around user-agent-defined behaviors

Reasonable caution should be exercised when changing implementation-specific
behavior of existing Web Platform APIs, to avoid breaking existing users. That
said, the leading proposal aims to align behavior around the existing behavior
of some implementers, and aims to better respect the user and developer intent.
It may still be reasonable for implementers to retain vestiges of old behavior
when the intent seems to rely on that behavior. For example, if developers
expect their implementer to clamp window placement within the current `Screen`
by using excessive bounds outside of any reasonable overall `Screen` space,
(e.g. "left=99999999"), then it may be reasonable to clamp window placement
within the current `Screen`, rather than the `Screen` nearest those coordinates.

### Open Questions

* Would changes to the existing synchronous methods break critical assumptions?
  * Do any sites want moveTo(0, 0) to always mean the current display's origin?
  * Do any sites expect open/move coordinates to be local to the current screen?
* How to handle requests for window bounds extending beyond a single display?
  * Is there value in supporting windows placements spanning multiple screens?
  * Suggest normative behavior for choosing the target display and clamping?
* Discuss window placement affordances around screenschange events?

## Privacy & Security

For an in-depth discussion on specific privacy and security concerns, see the
[responses to the W3C Security and Privacy Self-Review Questionnaire](https://github.com/webscreens/window-placement/blob/master/security_and_privacy.md),
and the [corresponding Screen Enumeration document](https://github.com/webscreens/screen-enumeration/blob/master/security_and_privacy.md)

TODO: Investigate concerns/mitigations for cross-screen placement/fullscreen.

[1]: https://developer.mozilla.org/en-US/docs/Web/API/Screen
[2]: https://developer.mozilla.org/en-US/docs/Web/API/Window
[3]: https://github.com/webscreens/screen-enumeration
