# Screen Enumeration / Window Placement

## Abstract

As it becomes more common to use more than one monitor, it becomes more important to give Web developers the tools to make their applications perform well across multiple displays.

The Window Placement API allows developers to configure the placement of one or more browser windows across one or more screens. Placement encompasses the position (x-, y-, z-coordinates) and size of the window, in addition to more complex behavior around dragging, aligning, and resizing windows.

The Screen Enumeration API gives developers access to a list of the available screens and the display properties of each screen. It provides the foundation for some Window Placement APIs and can be used to enhance others.

## Use cases

* **Slide show presentation using multiple screens**
  * Open the presentation, speaker notes, and presenter controls on different screens in fullscreen mode.
    ```js
    const screens = window.screens;

    // Option 1: Blow up multiple elements living in a single window.
    const presentation = document.querySelector("#presentation");
    const notes        = document.querySelector("#notes");
    const controls     = document.querySelector("#controls");

    presentation.requestFullscreen({ screen: screens[0] });
    notes.requestFullscreen({ screen: screens[1] });
    controls.requestFullscreen({ screen: screens[2] });

    // Option 2: Blow up multiple windows.
    window.open("/presentation", "presentation", "fullscreen", screens[0]);
    window.open("/notes", "notes", "fullscreen", screens[1]);
    window.open("/controls", "controls", "fullscreen", screens[2]);
    ```
  * Swap the presentation and notes (i.e. change the screen on which each window appears).
    ```js
    const presentationWindow = window.open("", "presentation");
    const notesWindow        = window.open("", "notes");
    presentationWindow.moveTo(notesWindow.screen);
    notesWindow.moveTo(presentationWindow.screen);

    // TODO: How would the size of the window be affected after the move?
    // TODO: window.rearrange({ window: screen }) ?
    // TODO: window.rearrange({ window: window1, screen: screen1, size: "100x100" },
    //                        { window: window2, screen: screen2, size: "fullscreen" }) ?
    ```
  * Move the speaker notes to a specific screen, not in fullscreen mode.
    ```js
    const screen = window.screens[0];
    const notesWindow = window.open("", "notes");
    notesWindow.moveTo(screen);
    notesWindow.resizeTo(100, 100);
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
        const screens = window.screens;
        const maxDashboardCount = 5;
        const usedScreenCount = Math.min(screens.length, maxDashboardCount);
        for (let screen = 1; screen < usedScreenCount; ++screen) {
          window.open(`/dashboard/${screen}`, `dashboard${screen}`, "",
                      screens[screen]);
        }
      });
    });
    ```
  * Starting the app restores all the dashboards' positions from the previous session.
    ```js
    // Service worker script
    function buildWindowConfig() {
      return {
        name: window.name,
        url: window.location,
        options: `left=${window.screenLeft},
                  top=${window.screenTop},
                  width=${window.outerWidth},
                  height=${window.outerHeight}`,
        screen: window.screen,
      };
    }

    function recordWindowConfig() {
      idb.open("db", 1)
         .transaction(["windowConfigs"], "readwrite")
         .objectStore("windowConfigs")
         .put(buildWindowConfig());
    }

    self.addEventListener("launch", event => {
      event.waitUntil(async () => {
        // Initialize IndexedDB database.
        const db = idb.open("db", 1, upgradeDb => {
          if (!upgradeDb.objectStoreNames.contains("windowConfigs")) {
            upgradeDb.createObjectStore("windowConfigs", { keyPath: "name" });
          }
        });

        // Retrieve dashboards' positions when app is launched.
        const configs = db.transaction(["windowConfigs"])
                          .objectStore("windowConfigs")
                          .getAll();
        for (let config : configs) {
          // Open each dashboard, assuming the user's screen setup hasn't changed.
          const dashboard = window.open(config.url, config.name, config.options,
              config.screen);

          // Record dashboard's position when it is closed.
          dashboard.addEventListener("beforeunload", recordWindowConfig);
        }
      });
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
  * Snap dashboards into place when moved, according to a pre-defined configuration of window positions.
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

The initial implementation will address only the slide show use case. All other use cases are left to future iterations of the API, though the initial API should be designed to accommodate them with minimal modifications.

## Proposal

### In scope

* Add `screens` property to `Window` interface
* Add `screen` to FullscreenOptions parameter of `Element.requestFullscreen()`
* Add `"fullscreen"` window feature to `Window.open()`
* Add `Screen` parameter to `Window.open()`
* Add `Screen` parameter to `Window.moveTo()`

### Future work

* Add `"alwaysOnTop"` window feature to `Window.open()`
* Create `"move"` Window event
* Create `"windowDrop"` Window event
* Create `"launch"` Service Worker event

## Privacy & Security
