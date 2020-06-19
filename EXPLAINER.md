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
* Allow elements to be shown fullscreen on any connected display
* Allow window placements on any connected display

### Possible future goals

Future goals may provide additional capabilities and more ergonomic APIs for web
applications to manage windows. See explorations of tentative future goals in
[additional_explorations.md](https://github.com/webscreens/window-placement/blob/master/additional_explorations.md).
* Offer more ergonomic APIs
* Extend window state and display mode APIs
* Surface events on window bounds, state, or display mode changes
* Support multiple fullscreen elements from a single document
* Extend placement affordances on transient user activation
* Allow placements that span two or more screens
* Support dependent or 'child' window types
* Allow sites to enumerate their windows

### Non-goals

* Open or move windows out of view from the user's screens
* Open or move windows across virtual workspaces/desktops
* Offer declarative window arrangements managed by the browser
* Open or move windows on remote displays connected to other devices
  * See the [Presentation](https://www.w3.org/TR/presentation-api/) and
    [Remote Playback](https://www.w3.org/TR/remote-playback/) APIs
* Capturing links in specific existing/new windows, etc.
  * See [Service Worker Launch Event](https://github.com/WICG/sw-launch) and
    [PWAs as URL Handlers](https://github.com/WICG/pwa-url-handler/blob/master/explainer.md)

## Proposal: Allow elements to be shown fullscreen on any connected display

This aspect of the proposal would add an optional Screen member to the optional
[`fullscreenOptions`](https://developer.mozilla.org/en-US/docs/Web/API/FullscreenOptions)
dictionary parameter of
[`Element.requestFullscreen()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/requestFullScreen).
See the proposed IDL change and a JS usage example below:

```js
dictionary FullscreenOptions {
  FullscreenNavigationUI navigationUI = "auto";

  // https://github.com/webscreens/window-placement
  Screen screen;
};
```

```js
// NEW: `getScreens()` provides requisite info; see the Screen Enumeration API.
const screens = await getScreens();
// Show the element fullscreen on an external display, if one exists.
// NEW: `screen` on `fullscreenOptions` for `requestFullscreen()`.
myElement.requestFullscreen({ screen: screens.find(s => !s.internal) });
```

As the `screen` dictionary member is not `required`, it is implicitly optional.
Callers can omit the member altogether to use the window's current screen, which
ensures backwards compatibility with existing usage. Callers can also explicitly
pass `undefined` for the same result. Passing `null` for this optional member is
not supported and will yield a TypeError, as is typical of modern web APIs.

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

## Proposal: Allow window placements on any connected display

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

```js
// NEW: `getScreens()` provides requisite info; see the Screen Enumeration API.
const otherScreen = (await getScreens()).find((s)=>{
    return s.availLeft != window.screen.availLeft ||
           s.availTop != window.screen.availTop;});
// Move the window to another screen (eg. user clicked "swap screens").
// NEW: `x` and `y` may be outside the window's current screen.
window.moveTo(otherScreen.availLeft + window.screenLeft,
              otherScreen.availTop + window.screenTop);
```

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

### Alternative new APIs

An alternative approach may be to introduce entirely new Window Placement APIs.
These could be built to solve frequent requests of modern web applications;
offering better ergonomics, asynchronous behavior, user permission affordances,
and more functionality. Thes could also be designed with multi-screen
environments, new window display modes, and related functionality in mind.

This could enable further deprecation of existing window placement APIs, which
have been prone to abuse and inconsistencies over time. There are several active
and forthcoming proposals to limit or deprecate existing window placement APIs.

Of course, aspects of new window placement APIs could prove undesirable in the
long term, similar to the existing APIs, and it may not be feasible to truly
deprecate old APIs, so this approach deserves caution. See some explorations of
possible new window placement API shapes in
[additional_explorations.md](https://github.com/webscreens/window-placement/blob/master/additional_explorations.md).

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

### Open questions

* Would changes to the existing synchronous methods break critical assumptions?
  * Do any sites expect open/move coordinates to be local to the current screen?
* Is there value in supporting windows placements spanning multiple screens?
  * Suggest normative behavior for choosing a target display and clamping?

## Privacy & Security

For an in-depth discussion on specific privacy and security concerns, see the
[responses to the W3C Security and Privacy Self-Review Questionnaire](https://github.com/webscreens/window-placement/blob/master/security_and_privacy.md).


and the [corresponding Screen Enumeration document](https://github.com/webscreens/screen-enumeration/blob/master/security_and_privacy.md)

TODO: Investigate concerns/mitigations for cross-screen placement/fullscreen.

[1]: https://developer.mozilla.org/en-US/docs/Web/API/Screen
[2]: https://developer.mozilla.org/en-US/docs/Web/API/Window
[3]: https://github.com/webscreens/screen-enumeration
