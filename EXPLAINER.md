# Window Placement

## Abstract

As it becomes more common to use more than one monitor, it becomes more
important to give Web developers the tools to make their applications perform
well across multiple displays.

This proposal allows developers to configure the placement of one or more
browser windows across one or more monitors. Placement encompasses the
position (x-, y-, z-coordinates) and size of the window, in addition to more
complex behavior around dragging, aligning, and resizing windows.

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

  // Option 1 (sync):
  //
  // NEW: `x` and `y` parameters of `moveTo()` accept coordinates outside of
  //      the window's current display.
  presentationWindow.moveTo(notesWindow.screen.availLeft,
                            notesWindow.screen.availTop);
  notesWindow.moveTo(presentationWindow.screen.availLeft,
                      presentationWindow.screen.availTop);

  // Option 2 (async):
  //
  // NEW: async `Window.setBounds()`
  presentationWindow.setBounds({
    x: notesWindow.screen.availLeft, y: notesWindow.screen.availTop
  });
  notesWindow.setBounds({
    x: presentationWindow.screen.availLeft,
    y: presentationWindow.screen.availTop
  });
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

The initial implementation will address only the slide show use case. All
[other use
cases](https://github.com/spark008/window-placement/blob/master/additional_use_cases.md)
are left to future iterations of the API, though the initial API should be
designed to accommodate them with minimal modifications.

## Proposal

### In scope

* Remove restrictions on `x` and `y` parameters of `Window.open()` and
`Window.moveTo()`
* Add `display` to FullscreenOptions parameter of `Element.requestFullscreen()`
* Add `"fullscreen"` window feature to `Window.open()`

### Future work

* Add async `setBounds()` on `Window` and `WindowClient`
* Add `"alwaysOnTop"` window feature to `Window.open()`
* Add `Display` parameter to `Clients.openWindow()`
* Create `"move"` Window event
* Create `"windowDrop"` Window event
* Create `"launch"` Service Worker event ([API explainer](https://github.com/WICG/sw-launch))

## Privacy & Security

TODO: Need to understand and investigate alleged security concerns about moving
windows offscreen.