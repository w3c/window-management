# Window Management on the Web - Many Windows Many Screens

## Introduction

This explainer aims to make mutli-screen experiences more flexible and convenient for users of web applications.

## Background

The [Window Management](https://github.com/w3c/window-management) API provides information about the device's connected screens, and allows web applications to open and position windows or request fullscreen on any of those screens.

A recent [enhancement](EXPLAINER_initiating_multi_screen_experiences.md) enabled web applications to [place fullscreen content on a specific screen](https://w3c.github.io/window-management/#usage-overview-place-fullscreen-content-on-a-specific-screen) and [place a new popup window on a separate specific screen](https://w3c.github.io/window-management/#usage-overview-place-windows-on-a-specific-screen), with a single [user interaction](https://html.spec.whatwg.org/multipage/interaction.html#tracking-user-activation).

A proposal for [Creating Fullscreen Popup Windows](EXPLAINER_fullscreen_popups.md) would enable web applications to open a new fullscreen window on a specific display, with a single user gesture. That proposal explicitly consumes [transient activation](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation), to limit fullscreen popup creation, even when the user agent is configured to permit popup window requests.

## Problem

Advanced web applications require additional flexibility, beyond recent enhancements and proposals, to conveniently initiate additional multi-screen experiences. Currently, each fullscreen request requires a separate user interaction, which adds undue user friction in some specific use cases.

## Use Cases

* Virtual Desktop app opens a fullscreen window for each screen
* Virtual Desktop app toggles fullscreen in existing windows for each screen
* Slideshow app presents fullscreen slides and speaker notes on separate screens
* Financial app launches a fullscreen dashboard encompassing multiple screens

## Goals

The goal of this document is to enable scripts with the multi-screen `window-management` permission to present content throughout the entirety of a multi-screen device's visual output, from a single transient user activation.

This builds on [Initiating Multi-Screen Experiences](EXPLAINER_initiating_multi_screen_experiences.md) to solve a more general pattern, beyond the highly specific scenario explored there. 

## Proposal

The proposal aims to enable executing a windowing action on each screen of a multi-screen device with a single user interaction, i.e.:
- Open a popup window [on each screen](https://w3c.github.io/window-management/#usage-overview-place-windows-on-a-specific-screen), potentially a new [fullscreen popup](EXPLAINER_fullscreen_popups.md)
- Make an existing window fullscreen [on each screen](https://w3c.github.io/window-management/#usage-overview-place-fullscreen-content-on-a-specific-screen), aided by [capability delegation](https://wicg.github.io/capability-delegation/spec.html#monkey-patch-to-fullscreen)
- Some mix thereof, e.g. one fullscreen and two popups, given three screens

### Extending User Activation

The design extends [Tracking User Activation](https://html.spec.whatwg.org/multipage/interaction.html#tracking-user-activation) with additional state specific to [powerful features](https://w3c.github.io/permissions/#dfn-powerful-feature). This enables a single [transient activation](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation) to permit multiple API requests pertaining to [`window-management`](https://w3c.github.io/window-management/#permissiondef-window-management), which would otherwise consume a singular state.

One possible shape for this proposal tracks an `activation set`, populated with types available for consumption by specific APIs (e.g. `default`, `window-management`, etc.), alongside the Window's [`activation timestamp`](https://html.spec.whatwg.org/multipage/interaction.html#last-activation-timestamp). The set is populated with a single `default` value on [`activation notification`](https://html.spec.whatwg.org/multipage/interaction.html#activation-notification).

The algorithm to [`consume user activation`](https://html.spec.whatwg.org/multipage/interaction.html#consume-user-activation) would take an optional parameter signifying the activation type to consume. When `window-management` is specified and transient activation has not expired, the algorithm would remove a `window-management` entry from the `activation set`, or remove a `default` entry, and then, iff the script is [allowed to use](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#allowed-to-use) `window-management`, append N-1 `window-management` entries, where N is the number of screens connected to the device.

### Extending Window Management APIs

The following API algorithms would would be modified to consume user activation state pertaining to [`window-management`](https://w3c.github.io/window-management/#permissiondef-window-management):
- [`Element.requestFullscreen()`](https://fullscreen.spec.whatwg.org/#dom-element-requestfullscreen)
- [`Window.open()`](https://html.spec.whatwg.org/multipage/nav-history-apis.html#dom-open)
-  [`Window.postMessage()`](https://html.spec.whatwg.org/multipage/web-messaging.html#window-post-message-steps) (when [initiating delegation](https://wicg.github.io/capability-delegation/spec.html#monkey-patch-to-html-initiating-delegation) of the [fullscreen](https://wicg.github.io/capability-delegation/spec.html#monkey-patch-to-html-initiating-delegation) capability)

### Cleanup redundant feature-specific spec changes
[Earlier work](EXPLAINER_initiating_multi_screen_experiences.md) added a very narrowly scoped internal slot on Window modeled after [last activation timestamp](https://html.spec.whatwg.org/multipage/#last-activation-timestamp), to conditionally permit a popup window after fullscreen is requested on a specific screen. That is no longer necessary with this more general proposal, and can be removed.

### Example Code

This feature can be used after the site successfully [requests detailed screen information](https://www.w3.org/TR/window-management/#usage-overview-screen-details). Script could now request fullscreen and open fullscreen popups on other screens, in a single event listener:

```js
initiateMultiScreenFullscreenButton.addEventListener('click', async () => {
  // Make the current window fullscreen on its current screen:
  const currentScreen = window.screenDetails.currentScreen;
  await document.documentElement.requestFullscreen({screen : currentScreen});

  // Open a fullscreen popup on each other screen.
  for (let s of window.screenDetails.screens) {
    window.open(url, '_blank', `popup,fullscreen,` +
                               `left=${s.availLeft},top=${s.availTop},` +
                               `width=${s.availWidth},height=${s.availHeight}`);
  }
});
```

Given a set of existing non-fullscreen popup windows opened to present content on each screen, script could delegate fullscreen to each of those windows from a single event listener:

```js
resumeMultiScreenFullscreenButton.addEventListener('click', async () => {
  // Delegate a fullscreen request to each existing cached window.
  for (let s of window.screenDetails.screens) {
    s.cachedPopup.postMessage('resumeFullscreen', {targetOrigin: origin, delegate: 'fullscreen'});
  }
});
```

A similar delegation technique would allow a single event listener to rearrange multiple existing fullscreen windows between screens.

### Open questions

#### Extending the UserActivation Interface

[The UserActivation interface](https://html.spec.whatwg.org/multipage/interaction.html#the-useractivation-interface) exposes activation state to script, and could be extended to offer a reasonable feature detection surface for the updated User Activation Tracking model.

## Security Considerations

This feature enables sites to perform multiple window management actions from a single user activation. This may exacerbate some documented Window Management [Security Considerations](https://www.w3.org/TR/window-management/#security), also considered in [Initiating Multi-Screen Experiences](EXPLAINER_initiating_multi_screen_experiences.md).

## Privacy Considerations

This feature does not expose any information to sites, and there are no privacy considerations to note beyond those already documented in the Window Management [Privacy Considerations](https://www.w3.org/TR/window-management/#privacy) section.

## Alternatives Considered

- Permit N `Window.open()` calls on a device with N screens
  - This does not satisfy the goal of also 'resuming' fullscreen
- Introduce a new API to support multi-screen fullscreen
  - This may be a desirable long-term path, but it is not nearly as tractable
- Gate window-management actions by activation, rather than consuming it
  - This allows unbounded actions from one gesture, which is more prone to abuse
- Tokenize access to individual screens
  - Enforces coherent initial placement, but popups can move without gesture
  - Introduces substantial complexity, particularly when delegating fullscreen
- Add more feature-specific internal slots, like [earlier work](EXPLAINER_initiating_multi_screen_experiences.md)
  - Less encapsulated and more complex than enahncing user activation tracking