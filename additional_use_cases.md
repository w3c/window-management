# Additional Use Cases

The following use cases play a role in shaping the API design, but will be
implemented in future iterations.

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
  * Open dependent/child windows that the OS moves with a parent window.
    ```js
    const palette1 = window.open("/palette/1", "palette1", "dependent=true");
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
