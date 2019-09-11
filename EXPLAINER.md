# Window Placement

## Abstract

As it becomes more common to use more than one monitor, it becomes more
important to give Web developers the tools to make their applications perform
well across multiple displays with differing properties.

This proposal allows developers to configure the arrangement of browser windows
on devices with multiple monitors. Beyond the intial window placement and state
(eg. maximized, fullscreen, normal/restored), this proposal aims to build tools
around the movement, sizing, and state changes of existing windows.

Many parts of the Window Placement API either depend upon, or are enhanced
by, access to the end user's screen configuration. Such access may be made
available through a [Screen Enumeration
API](https://github.com/spark008/screen-enumeration).

## Use case

**Slide show presentation using multiple displays**
* Open the presentation, speaker notes, and presenter controls on different
  displays in fullscreen mode.
  * Scenario 1: No service worker
    ```js
    const displays = await ScreenEnumerationAPI.displays();

    // Option 1: Blow up multiple elements living in a single window.
    const presentation = document.querySelector("#presentation");
    const notes        = document.querySelector("#notes");
    const controls     = document.querySelector("#controls");

    // NEW: Add `display` parameter on `requestFullscreen()`.
    presentation.requestFullscreen({ display: displays[0] });
    notes.requestFullscreen({ display: display[1] });
    controls.requestFullscreen({ display: display[2] });

    // Option 2: Blow up multiple windows.
    //
    // NEW: `left` and `top` window feature parameters accept coordinates
    //      outside of the window's current display.
    window.open(
      "/presentation",
      "presentation",
      `fullscreen,left=${displays[0].availLeft},top=${displays[0].availTop}`
    );
    window.open(
      "/notes",
      "notes",
      `fullscreen,left=${displays[1].availLeft},top=${displays[1].availTop}`
    );
    window.open(
      "/controls",
      "controls",
      `fullscreen,left=${displays[2].availLeft},top=${displays[2].availTop}`
    );
    ```
  * Scenario 2: Using service worker
    ```js
    // Service worker script
    const displays = await ScreenEnumerationAPI.displays();

    // NEW: Add a `windowOptions` object parameter to `openWindow()` that
    //      accepts `x`, `y`, `width`, `height`, and `state`.
    async function launchPresentation() {
      await clients.openWindow("/presentation", {
        x: displays[0].availLeft,
        y: displays[0].availTop,
        state: "fullscreen",
      });
      await clients.openWindow("/notes", {
        x: displays[1].availLeft,
        y: displays[1].availTop,
        width: 1000,
        height: 300,
      });
      await clients.openWindow("/controls", {
        x: displays[2].availLeft,
        y: displays[2].availTop,
        width: 500,
        height: 500,
      });
    }
    ```
* Swap the presentation and notes (i.e. change the monitor on which each
  window appears).
  ```js
  const presentationWindow = window.open("", "presentation");
  const notesWindow        = window.open("", "notes");
  notesBounds = { x: notesWindow.screen.availLeft,
                  y: notesWindow.screen.availTop,
                  width: notesWindow.screen.availWidth,
                  height: notesWindow.screen.availHeight };
  presentationBounds = { x: presentationWindow.screen.availLeft,
                         y: presentationWindow.screen.availTop,
                         width: presentationWindow.screen.availWidth,
                         height: presentationWindow.screen.availHeight };

  // Option 1 (sync):
  //
  // NEW: `x` and `y` parameters of `moveTo()` accept coordinates outside of
  //      the window's current display.
  presentationWindow.moveTo(notesBounds.x, notesBounds.y);
  notesWindow.moveTo(presentationBounds.x, presentationBounds.y);
  presentationWindow.resizeTo(notesBounds.width, notesBounds.height);
  notesWindow.resizeTo(presentationBounds.width, presentationBounds.height);

  // Option 2 (async):
  //
  // NEW: async `Window.setBounds()`
  presentationWindow.setBounds(notesBounds);
  notesWindow.setBounds(presentationBounds);
  ```
* Move the speaker notes to a specific display, not in fullscreen mode.
  * Scenario 1: No service worker
    ```js
    const notesWindow = window.open("", "notes");
    const display = ScreenEnumerationAPI.displays[0];

    // Option 1: Move and resize in 2 steps
    notesWindow.moveTo(display.availLeft + 10, display.availTop + 10);
    notesWindow.resizeTo(100, 100);

    // Option 2: Move and resize in a single step
    //
    // NEW: async `Window.setBounds()`
    await notesWindow.setBounds({
      x: display.availLeft + 10,
      y: display.availTop + 10,
      width: 100,
      height: 100
    });
    ```
  * Scenario 2: Using service worker
    ```js
    // Service worker script
    async () => {
      const notesWindowClient = await clients.get(1);
      const display = ScreenEnumerationAPI.displays[0];

      // Option 1: Move and resize in 2 steps
      //
      // NEW: async `WindowClient.moveTo()` and async `WindowClient.resizeTo()`
      await notesWindowClient.moveTo(display.availLeft + 10,
                                      display.availTop + 10);
      await notesWindowClient.resizeTo(100, 100);

      // Option 2: Move and resize in a single step
      //
      // NEW: async `WindowClient.setBounds()`
      await notesWindow.setBounds({
        x: display.availLeft + 10,
        y: display.availTop + 10,
        width: 100,
        height: 100
      });
    }
    ```

## Goals / Non-goals

The goal of this proposal is to provide the basic tools for developers to
present content across a device's set of connected displays.

[Other use cases](https://github.com/spark008/window-placement/blob/master/additional_use_cases.md)
are left to future iterations of the API, though the initial API should be
designed to accommodate them with minimal modifications.

### Current goals

* Support creation of windows across the set of connected displays.
* Support movement of windows between the set of connected displays.

### Future goals

* Support the opening of windows in a fullscreen or always-on-top state.
* Support changes of the window state (eg. maximize, restore).
* Support observation of changes in window bounds and state.
* Support the creation and management of dependent or 'child' windows.

### Non-goals

* Provide a high-level API for the automated arrangement of window groupings.

## Proposal

### In scope

Option A (async approach, **preferred**):
* Add `windowOptions` parameter to `Client.openWindow()`.
  * windowOptions would accept these properties:
    * **`x`**: The x-coordinate (left) of the window to open in screen space.
    * **`y`**: The y-coordinate (top) of the window to open in screen space.
    * **`width`**: The width of the window to open.
    * **`height`**: The height of the window to open.
    * **`state`**: The state of the window, with support for these values:
      * **`normal`**: Normal window state (not maximized or fullscreen)
      * **`fullscreen`**: Window content fills the display without a frame.
      * **`maximized`**: The window content occupys the full screen space.
    * **`type`**: The type of the window, with support for these values:
      * **`normal`**: A normal tabbed browser window.
      * **`fullscreen`**: A popup window with minimal frame elements.

* Add async `setBounds()` on `Window` and `WindowClient` that accepts `x`, `y`,
`width`, `height`, and `state`

Option B (sync approach):
* Remove restrictions on `x` and `y` parameters of `Window.open()` and
`Window.moveTo()`
* Add/restore `"fullscreen"` window feature to `Window.open()` ([currently deprecated](https://developer.mozilla.org/en-US/docs/Web/API/Window/open#Window_functionality_features))

### Future work

* Add `display` to FullscreenOptions parameter of `Element.requestFullscreen()`
* Add `"alwaysOnTop"` feature to `Window.open()` or `clients.openWindow()`
* Add `Display` parameter to `Clients.openWindow()`
* Add `"onmove"`, `"onwindowdrag"`, and `"onwindowdrop"` Window events
* Add `"onwindowstate"` Window event for state changes
* Add `"launch"` Service Worker event ([API explainer](https://github.com/WICG/sw-launch))
* Add client. Enumerate existing windows

### Open Questions

* Should windowOptions use inerWidth, outerWidth, or both? (same for height)
* What window states and types are the most important for developers?
* Could permission for screen enumeration and window placement be combined?
* Would changes to the existing synchronous methods break crticial assumptions?

## Privacy & Security

TODO: Need to understand and investigate alleged security concerns about moving
windows between displays.