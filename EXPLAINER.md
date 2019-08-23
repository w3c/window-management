# Window Placement

## Abstract

As it becomes more common to use more than one monitor, it becomes more
important to give Web developers the tools to make their applications perform
well across multiple displays.

This proposal allows developers to configure the placement of one or more
browser windows across one or more screens. Placement encompasses the
position (x-, y-, z-coordinates) and size of the window, in addition to more
complex behavior around dragging, aligning, and resizing windows.

Many parts of the Window Placement API either depend upon, or are enhanced
by, access to the end user's screen configuration. Such access may be made
available through a [Screen Enumeration
API](https://github.com/spark008/screen-enumeration), which may use a screen
picker UI built into the browser.

## Use cases

* **Slide show presentation using multiple screens**
  * Open the presentation, speaker notes, and presenter controls on different
    screens in fullscreen mode.
    * Scenario A: No service worker
      ```js
      const screens = (async () => {
        try {
          return await ScreenEnumerationAPI.screens();
        } catch (e) {
          return "oh noes";
        }
      })();

      // Option 1: Blow up multiple elements living in a single window.
      //
      // NEW: Add `screen` parameter on `requestFullscreen()`.
      const presentation = document.querySelector("#presentation");
      const notes        = document.querySelector("#notes");
      const controls     = document.querySelector("#controls");

      presentation.requestFullscreen({ screen: screens[0] });
      notes.requestFullscreen({ screen: screens[1] });
      controls.requestFullscreen({ screen: screens[2] });

      // Option 2: Blow up multiple windows.
      //
      // NEW: `left` and `top` window feature parameters take absolute screen
      //      coordinates, rather than display-relative coordinates.
      window.open(
        "/presentation",
        "presentation",
        `fullscreen,left=${screens[0].availLeft},top=${screens[0].availTop}`
      );
      window.open(
        "/notes",
        "notes",
        `fullscreen,left=${screens[1].availLeft},top=${screens[1].availTop}`
      );
      window.open(
        "/controls",
        "controls",
        `fullscreen,left=${screens[2].availLeft},top=${screens[2].availTop}`
      );
      ```
    * Scenario 2: Using service worker
      ```js
      // Service worker script
      const screens = await ScreenEnumerationAPI.screens();

      // NEW: Add a `windowOptions` object parameter to `openWindow()` that
      //      accepts `x`, `y`, `width`, `height`, and `state`.
      async function launchPresentation() {
        await clients.openWindow("/presentation", {
          x: screens[0].availLeft,
          y: screens[0].availTop,
          state: "fullscreen",
        });
        await clients.openWindow("/notes", {
          x: screens[1].availLeft,
          y: screens[1].availTop,
          width: 1000,
          height: 300,
        });
        await clients.openWindow("/controls", {
          x: screens[2].availLeft,
          y: screens[2].availTop,
          width: 500,
          height: 500,
        });
      }
      ```
  * Swap the presentation and notes (i.e. change the screen on which each window
    appears).
    ```js
    const presentationWindow = window.open("", "presentation");
    const notesWindow        = window.open("", "notes");

    // NEW: `x` and `y` parameters of `moveTo()` take absolute screen
    //      coordinates, rather than display-relative coordinates.
    presentationWindow.moveTo(notesWindow.screen.availLeft,
                              notesWindow.screen.availTop);
    notesWindow.moveTo(presentationWindow.screen.availLeft,
                       presentationWindow.screen.availTop);
    ```
  * Move the speaker notes to a specific screen, not in fullscreen mode.
    * Scenario 1: No service worker
      ```js
      const screen = window.screens[0];
      const notesWindow = window.open("", "notes");

      // Option 1: Move and resize in 2 steps
      notesWindow.moveTo(notesWindow.screen.availLeft + 10,
                         notesWindow.screen.availTop + 10);
      notesWindow.resizeTo(100, 100);

      // Option 2: Move and resize in a single step
      ```
    * Scenario 2: Using service worker
      ```js
      // Service worker script
      async () => {
        const screen = self.screens[0];
        const notesWindowClient = await clients.get(1);

        // Option 1: Move and resize in 2 steps
        await notesWindow.moveTo(notesWindowClient.screen.availLeft + 10,
                                 notesWindowClient.screen.availTop + 10);
        await notesWindowClient.resizeTo(100, 100);

        // Option 2: Move and resize in a single step
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

* Add `screen` to FullscreenOptions parameter of `Element.requestFullscreen()`
* Add `"fullscreen"` window feature to `Window.open()`
* Add `Screen` parameter to `Window.open()`
* Add `Screen` parameter to `Window.moveTo()`

### Future work

* Add `"alwaysOnTop"` window feature to `Window.open()`
* Add `Screen` parameter to `Clients.openWindow()`
* Create `"move"` Window event
* Create `"windowDrop"` Window event
* Create `"launch"` Service Worker event ([API explainer](https://github.com/WICG/sw-launch))

## Privacy & Security
