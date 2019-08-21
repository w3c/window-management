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
* **Professional image editing tools with floating palettes**
  * Always keep the palettes on top of the main editor.
    ```js
    window.open("/palette", "palette", "alwaysOnTop");
    ```
  * Synchronously move the palettes when the main editor moves.
    ```js
    const palette1 = window.open("/palette/1", "palette1", "alwaysOnTop");
    const palette2 = window.open("/palette/2", "palette2", "alwaysOnTop");
    window.addEventListener("move", event => {
      palette1.moveBy(event.deltaX, event.deltaY);
      palette2.moveBy(event.deltaX, event.deltaY);
    });
    ```
* **Finance applications with multiple dashboards**
  * Starting the app opens all the dashboards across multiple screens.
    ```js
    // Service worker script
    self.addEventListener("launch", event => {
      event.waitUntil(async () => {
        const screens = self.screens;
        const maxDashboardCount = 5;
        const usedScreenCount = Math.min(screens.length, maxDashboardCount);
        for (let screen = 1; screen < usedScreenCount; ++screen) {
          await clients.openWindow(`/dashboard/${screen}`, screens[screen]);
        }
      });
    });
    ```
  * Starting the app restores all the dashboards' positions from the previous
    session.
    ```js
    // Service worker script

    self.addEventListener("launch", event => {
      event.waitUntil(async () => {
        // Initialize IndexedDB database.
        const db = idb.open("db", 1, upgradeDb => {
          if (!upgradeDb.objectStoreNames.contains("windowConfigs")) {
            upgradeDb.createObjectStore("windowConfigs", { keyPath: "name" });
          }
        });

        // Retrieve preferred dashboard configurations.
        const configs = db.transaction(["windowConfigs"])
                          .objectStore("windowConfigs")
                          .getAll();
        for (let config : configs) {
          // Open each dashboard, assuming the user's screen setup hasn't changed.
          const dashboard = await clients.openWindow(config.url, config.screen);
          dashboard.postMessage(config.options);
        }
      });
    });

    // Record the latest configuration relayed by a dashboard that was just closed.
    self.addEventListener("message", event => {
      idb.open("db", 1)
         .transaction(["windowConfigs"], "readwrite")
         .objectStore("windowConfigs")
         .put(event.data);
    });
    ```
    ```js
    // Dashboard script

    window.name = window.location;

    // Configure dashboard according to preferences relayed by the Service Worker.
    window.addEventListener("message", event => {
      window.moveTo(event.data.left, event.data.top);
      window.resizeTo(event.data.width, event.data.height);
    });

    // Send dashboard's configuration to the Service Worker just before closing.
    window.addEventListener("beforeunload", event => {
      const windowConfig = {
        name: window.name,
        url: window.location,
        options: {
          left: window.screenLeft,
          top: window.screenTop,
          width: window.outerWidth,
          height: window.outerHeight,
        },
        screen: window.screen,
      };
      navigator.serviceWorker.controller.postMessage(windowConfig);
    });
    ```
  * Align dashboards relative to each other, or to the screen.
    ```js
    // Horizontally center dashboards 1 and 2, relative to each other.
    //   ______________           ______________
    //  | ____         |         |              |
    //  ||   x|   ____ |         |         ____ |
    //  ||    |  |   x||         | ____   |   x||
    //  || 1  |  |    ||         ||   x|  |    ||
    //  ||____|  |    ||   --->  ||    |  |    ||
    //  |        |  2 ||         || 1  |  |  2 ||
    //  |        |    ||         ||____|  |    ||
    //  |        |____||         |        |____||
    //  |______________|         |______________|
    //
    // NOTE: No new APIs used.
    function horizontallyCenter(dashboards) {
      let maxHeight = 0;
      let newCenter;
      for (dashboard in dashboards) {
        if (dashboard.outerHeight > maxHeight) {
          maxHeight = dashboard.outerHeight;
          newCenter = dashboard.screenY + dashboard.outerHeight / 2;
        }
      }
      for (dashboard in dashboards) {
        const oldCenter = dashboard.screenY + dashboard.outerHeight / 2;
        dashboard.moveBy(0, oldCenter - newCenter);
      }
    }
    ```
  * Snap dashboards into place when moved, according to a pre-defined
    configuration of window positions.
    ```js
    // Reposition/resize window into the nearest left/right half of the screen when
    // programatically moved or manually dragged then dropped.
    window.addEventListener("windowDrop", event => {
      const windowCenter = window.screenLeft + window.outerWidth / 2;
      const screenCenter = window.screen.availLeft + window.screen.availWidth / 2;
      const newLeft = (windowCenter < screenCenter) ? 0 : screenCenter;
      window.moveTo(newLeft, 0);
      window.resizeTo(window.screen.width, window.screen.height);
    });
    ```
* **Small form-factor applications, e.g. calculator, mini music player**
  * Launch the app with specific (or bounded) dimensions.
    ```js
    // Service worker script
    self.addEventListener("launch", event => {
      event.waitUntil(async () => {
        // At most 800px wide.
        const width = Math.min(window.screen.availWidth * 0.5, 800);
        // At least 200px tall.
        const height = Math.max(window.screen.availHeight * 0.3, 200);

        window.resizeTo(width, height);
      });
    });
    ```

## Goals / Non-goals

The initial implementation will address only the slide show use case. All other
use cases are left to future iterations of the API, though the initial API
should be designed to accommodate them with minimal modifications.

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
