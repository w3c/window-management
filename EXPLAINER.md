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

## Use case

Given information about connected screens from the [Screen Enumeration API][3]:
```js
const screens = await navigator.screen.getScreens();
```
Consider use cases like those described below and in the
[additional use cases](https://github.com/spark008/window-placement/blob/master/additional_use_cases.md)
document.

* **Slide show presentation with a laptop screen and a projector**
  * Present slides on the projector, open speaker notes/controls on the laptop
* **Finance applications with multiple windows spread over multiple displays**
  * Launch a dashboard that opens a set of windows across multiple displays
  * User interacts with dashboard controls to move windows between displays
* **Media and medical applications target specialized display hardware**
  * Precise content is shown on displays with high resolutions or color depths

**Slide show presentation and speaker notes using multiple displays**
* Open the slides and notes on separate displays in fullscreen mode.
  * Scenario 1: No service worker
    ```js
    // NEW: `left` and `top` may be outside the window's current display.
    const slides = window.open("/slides", "slides", `left=${screens[1].left},top=${screens[1].top}`);
    const notes = window.open("/notes", "notes", `left=${screens[0].left},top=${screens[0].top}`);
    slides.document.body.requestFullscreen();
    notes.document.body.requestFullscreen();
     ```
  * Scenario 2: Using service worker
    ```js
    // NEW: Add a `windowOptions` object parameter to `openWindow()` that
    //      accepts `left`, `top`, `width`, `height`.
    await clients.openWindow("/slides", {
      left: screens[1].left,
      top: screens[1].top,
    }).then(requestDocumentBodyFullscreen(WindowClient));
    await clients.openWindow("/notes", {
      left: screens[0].left,
      top: screens[0].top,
    }).then(requestDocumentBodyFullscreen(WindowClient));
    ```
* Swap the slides and notes windows between displays.
  ```js
  // Option 1 (sync):
  // NEW: `x` and `y` may be outside the window's current screen.
  slidesWindow.moveTo(screens[0].left, screens[0].top);
  notesWindow.moveTo(screens[1].left, screens[1].top);
  slidesWindow.document.body.requestFullscreen();
  notesWindow.document.body.requestFullscreen();

  // Option 2 (async):
  const slidesBounds = { x: screens[0].left, y: screens[0].top,
                         width: screens[0].width, height: screens[0].height };
  const notesBounds = { x: screens[1].left, y: screens[1].top,
                        width: screens[1].width, height: 500 };
  // NEW: async `Window.setBounds()` accepts location and size
  await slidesWindow.setBounds(slidesBounds);
  await notesWindow.setBounds(notesBounds);
  ```

TODO: Show how these APIs might evolve with some possible future work, to ensure
that intermediate proposals make sense in an evolving landscape.

## Goals / Non-goals

The goal of this proposal is to provide the basic tools for developers to
place and move content across a device's set of available connected displays.
[Other use cases](https://github.com/spark008/window-placement/blob/master/additional_use_cases.md)
may depend on future iterations of the API, though the initial API should be
designed to accommodate them with minimal modifications.

### Current goals

* Open application windows on any connected display
* Move application windows to any connected display

### Future goals

* Extend window state control APIs (e.g. maximize, restore, fullscreen)
* Refine API shapes (e.g. add optional `windowOptions` to window.open())
* Surface events when a window's bounds or state changes
* Extend window creation and management for dependent or 'child' window types
* Offer window frame appearance controls

### Non-goals

* Open or move windows out of view from the user's screens
* Open or move windows across virtual workspaces/desktops
* Offer declarative window arrangements managed by the browser
* Support explicit z-ordering, such as an `"alwaysOnTop"` window state
* Open or move windows on remote displays connected to other devices
  * See the
  [Presentation](https://www.w3.org/TR/presentation-api/)
  and [Remote Playback](https://www.w3.org/TR/remote-playback/) APIs

## Proposal

### Leading Proposal

The current leading proposal is to accept coordinates located on any `Screen` in
the [`Window.open()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/open)
and [`Window.moveTo()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/moveTo)
methods.

The [`Window`][2] and [`Screen`][1] interfaces generally already use coordinates
in the overall screen space, relative to the primary display (see the
[Screen Enumeration API][3] for more details). Additionally, some implementers
already support placement on, and movement to, screens other than the one
hosting the `Window` executing the script. However, API behavior is unspecified
and implementations are inconsistent. Some implementations clamp the requested
window bounds to be within the same `Screen` as the host `Window`*. It may be
appropriate to standardize the API behavior or simply provide non-normative
notes encouraging more consistent support for increased interoperability.

\* TODO: Provide a more complete description of behaviors and discrepancies.

### Alternative or Supplemental Proposal

An alternative or supplemental proposal is to extend the asynchronous
`Client.openWindow()` method with a `windowOptions` parameter with these
properties:
* **`left`**: The x-coordinate of the window to open in screen space
* **`top`**: The y-coordinate of the window to open in screen space
* **`width`**: The width of the window to open
* **`height`**: The height of the window to open

The asynchronous pattern affords implementers the opportunity to check for or
request permission before acting, without stopping script execution.
Additionally, offering a new API surface would have no impact on existing users.

It may then make sense to offer an asynchronous `setBounds()` method on the
`Window` and `WindowClient` interfaces that accepts `x`, `y`, `width`, `height`;
similarly restricting usage to new callers and allowing for permission checks
or requests.

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

### Respecting Intents Around Deprecated Implementer-Specific Behavior

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

### Future Work

* Restore `"fullscreen"` feature functionality for `Window.open()`, currently
  [deprecated](https://developer.mozilla.org/en-US/docs/Web/API/Window/open#Window_functionality_features)
  * Adding this (and window features like `"maximized"`) to `Window.open()` may
    solve existing use cases around desired window state and types, but such
    changes should be considered with some cautionary context. This API shape is
    unfortunate with a loosely defined string format and a set of unstandardized
    features with a mixed history of abuse and deprecation. It is also a
    potentially more abusable synchronous API, making permission checks and
    requests more difficult.
* Extend the `windowOptions` parameter to `openWindow()` with these properties:
  * **`state`**: The window state, with support for these values:
    * **`normal`**: Normal window state (not maximized nor fullscreen)
    * **`fullscreen`**: Window content fills the display without a frame
    * **`maximized`**: Window content occupies the full screen space
  * **`type`**: The window type, with support for these values:
    * **`normal`**: A normal tabbed browser window
    * **`popup`**: A popup window with minimal frame elements
  * **`screen`**: The desired host screen (redundant with screen-space x,y?)
* Add `screen` parameter to `Element.requestFullscreen()`
* Add `"onmove"`, `"onwindowdrag"`, and `"onwindowdrop"` Window events
* Add `"onwindowstate"` Window event for state changes
* Add `"snapLeft"`, `"snapRight"`, and other OS/WM **`state`** enum values
* Add `"app"` or other possible **`type`** enum values
* Add `"launch"` Service Worker event ([API explainer](https://github.com/WICG/sw-launch))
* Add `client.Enumerate()` to list existing windows

### Open Questions

* Should windowOptions use innerWidth, outerWidth, or both? (same for height)
* What window states and types are the most important for developers?
* Would changes to the existing synchronous methods break critical assumptions?
  * Do any sites want moveTo(0, 0) to always mean the current display's origin?
* How to handle requests for window bounds extending beyond a single display?
  * Considerations around title bar visibility and minimum sizes.

## Privacy & Security

* TODO: Investigate concerns about moving windows between displays.
* Should permission for screen enumeration and window placement be combined?
* Any privacy concerns beyond those considered for the
  [Screen Enumeration API](https://github.com/spark008/screen-enumeration/blob/master/security_and_privacy.md)?

[1]: https://developer.mozilla.org/en-US/docs/Web/API/Screen
[2]: https://developer.mozilla.org/en-US/docs/Web/API/Window
[3]: https://github.com/spark008/screen-enumeration
