# Window Management on the Web - Many Windows Many Screens

## Introduction

This proposal introduces a Window Management API enhancement that would allow web applications to initiate new multi-screen experiences from a single user activation.

## Background

The [Window Management](https://github.com/w3c/window-management) API provides information about the device's connected screens, and allows web applications to open and position windows or request fullscreen on any of those screens.

A recent [enhancement](EXPLAINER_initiating_multi_screen_experiences.md) enabled web applications to [place fullscreen content on a specific screen](https://w3c.github.io/window-management/#usage-overview-place-fullscreen-content-on-a-specific-screen) *and* [place a new popup window on a separate specific screen](https://w3c.github.io/window-management/#usage-overview-place-windows-on-a-specific-screen), with a single [user activation signal](https://html.spec.whatwg.org/multipage/interaction.html#tracking-user-activation).

A proposal for [Creating Fullscreen Popup Windows](EXPLAINER_fullscreen_popups.md) would enable web applications to open a new fullscreen window on a specific display, with a single user gesture. That proposal explicitly consumes [transient activation](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation), to prohibit abusive creation of unlimited fullscreen popups from a single user activation, even when the user agent is configured to permit popup window requests.

## Problem

Advanced web applications request additional flexibility, beyond recent enhancements and proposals, to conveniently initiate additional multi-screen experiences. The most notable example is starting a multi-screen fullscreen virtual desktop session.

The common theme among several requests is the ability to execute a windowing action on each screen of a multi-screen device with a single user interaction, i.e.:
- Open a popup window on each screen, potentially a new [fullscreen popup](EXPLAINER_fullscreen_popups.md)
- Make an existing window fullscreen on each screen
- Some mix thereof, e.g. one fullscreen and two popups, given three screens

While these capabilities are available to other application development platforms, web applications are limited to making a single fullscreen window per user interaction.

## Use Cases

* Virtual Desktop app opens a fullscreen window for each screen
* Virtual Desktop app toggles fullscreen in existing windows for each screen
* Slideshow app presents fullscreen slides and speaker notes on separate screens
* Financial app launches a fullscreen dashboard encompassing multiple screens

## Goals

The goal of this document is to explore ways to safely enable multiple windowing actions from a single user gesture on multi-screen devices. This builds on [Initiating Multi-Screen Experiences](EXPLAINER_initiating_multi_screen_experiences.md) to solve a more general problem, beyond the highly specific usage pattern explored there.

In particular, the goal here is to enable scripts with the multi-screen `window-management` permission to present content throughout the entirety of a multi-screen device's visual output, from a single transient user activation.

## Proposal

The design builds on Window Management and User Activation APIs, decomposing transient user activation signals into multiple consumable tokens, and leveraging prior work that enables sites to place [popups](https://w3c.github.io/window-management/#usage-overview-place-windows-on-a-specific-screen) and [fullscreen](https://w3c.github.io/window-management/#usage-overview-place-fullscreen-content-on-a-specific-screen) on specific screens, and to [delegate fullscreen](https://wicg.github.io/capability-delegation/spec.html#monkey-patch-to-fullscreen) requests to another window.

In particular, the proposal is to extend `Element.requestFullscreen()`, `Window.open()`, `Window.postMessage()` and `Tracking User Activation` algorithms to conditionally permit multiple window management requests from a single transient user activation.

The number of window management requests honored for a single user activation is equal to the number of extended screens of the device.

### Extending User Activation

One possible shape for this proposal tracks an `activation set`, populated with types available for consumption by specific APIs (e.g. `default`, `window-management`, etc.), alongside the Window's `[activation timestamp](https://html.spec.whatwg.org/multipage/interaction.html#last-activation-timestamp)`. The set is populated with a single `default` value on `[activation notification](https://html.spec.whatwg.org/multipage/interaction.html#activation-notification]`.

The algorithm to `[consume user activation](https://html.spec.whatwg.org/multipage/interaction.html#consume-user-activation)` would take an optional parameter signifying the activation type to consume. When `window-management` is specified and transient activation has not expired, the algorithm would remove a `window-management` entry from the `activation set`, or remove a `default` entry, and then, iff `window-management` permission is granted, append N-1 `window-management` entries, where N the number of screens connected to the device.

### Extending Window Management APIs

APIs pertaining to `window-management` would specify the `window-management` activation type to consume, notably:
- `Element.requestFullscreen()`
- `Window.open()`
- `Window.postMessage()` containing a [fullscreen delegation](https://wicg.github.io/capability-delegation/spec.html#monkey-patch-to-fullscreen)

### Example Code

This feature can be used after the site successfully [requests detailed screen information](https://www.w3.org/TR/window-management/#usage-overview-screen-details). Script could now request fullscreen, and open fullscreen popups on other displays, or make existing popups fullscreen on other displays, in a single event listener:

```js
initiateMultiScreenFullscreenButton.addEventListener('click', async () => {
  // Make the current window fullscreen on its current screen:
  const currentScreen = window.screenDetails.currentScreen;
  await document.documentElement.requestFullscreen({screen : currentScreen});

  // Open a fullscreen popup on each other display.
  for (let s of window.screenDetails.screens) {
    window.open(url, '_blank', `popup,fullscreen,` +
                               `left=${s.availLeft},top=${s.availTop},` +
                               `width=${s.availWidth},height=${s.availHeight}`);
  }
});
```

Given a set of existing non-fullscreen popup windows opened to present content on each screen, script could now delegate fullscreen to each of those windows from a single event listener:

```js
resumeMultiScreenFullscreenButton.addEventListener('click', async () => {
  // Delegate a fullscreen request to each existing cached window.
  for (let s of window.screenDetails.screens) {
    s.cachedPopup.postMessage('resumeFullscreen', {targetOrigin: origin, delegate: 'fullscreen'});
  }
});
```

A similar delegation technique would allow a single event listener to rearrange existing fullscreen windows between screens.

### Open questions

#### Extending the UserActivation Interface

[The UserActivation interface](https://html.spec.whatwg.org/multipage/interaction.html#the-useractivation-interface) exposes activation state to script, and could be extended to offer a reasonable feature detection surface for the updated User Activation Tracking model.

## Security Considerations

This feature enables sites to perform multiple window management actions from a single user activation. This may exacerbate some documented Window Management [Security Considerations](https://www.w3.org/TR/window-management/#security), also considered in [Initiating Multi-Screen Experiences](EXPLAINER_initiating_multi_screen_experiences.md).

## Privacy Considerations

This feature does not expose any information to sites, and there are no privacy considerations to note beyond those already documented in the Window Management [Privacy Considerations](https://www.w3.org/TR/window-management/#privacy) section.

## Alternatives Considered

- Permit N `Window.open()` calls on a device with N screens
  - This doesn't satisfy the goal of also 'resuming' fullscreen
- Introduce a new API to support multi-screen fullscreen
  - This may be a desirable long-term path, but it is not nearly as tractable
- Gate window-management actions by activation, rather than consuming it
  - This allows unbounded actions from one gesture, which is more abusable
- Tokenize access to individual screens
  - Enforces initial placement protection, but popups can move without gesture
  - Introduces substantial complexity, particularly when delegating fullscreen
