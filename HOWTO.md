# How to use the Window Management API

The Window Management API is currently available in Chrome:
- Chrome 100+ enables Window Management APIs by default.
- Chrome 93+ supports Window Management APIs with either one of these flags enabled:
  - chrome://flags#enable-experimental-web-platform-features
  - `chrome --enable-blink-features=WindowPlacement`

Try these basic API demos, which request permission to use info about your screens to open and place windows:
- https://michaelwasserman.github.io/window-placement-demo
- https://developer.chrome.com/articles/window-management/ has a demo: https://window-placement.glitch.me

Here is an example of how to use the API:

```javascript
async function main() {
  // Run feature detection.
  if (!('getScreenDetails' in window)) {
    console.log('The Window Manamgenet API is not supported.');
    return;
  }
  // Detect the presence of extended screen areas.
  if (window.screen.isExtended) {
    // Request extended screen information.
    const screenDetails = await window.getScreenDetails();

    // Find the primary screen, show some content fullscreen there.
    const primaryScreen = screenDetails.screens.find(s => s.isPrimary);
    document.documentElement.requestFullscreen({screen : primaryScreen});

    // Find a different screen, fill its available area with a new window.
    const otherScreen = screenDetails.screens.find(s => s != primaryScreen);
    window.open(url, '_blank', `left=${otherScreen.availLeft},` +
                               `top=${otherScreen.availTop},` +
                               `width=${otherScreen.availWidth},` +
                               `height=${otherScreen.availHeight}`);
  } else {
    // Arrange content within the traditional single-screen environment.
  }
};
```

## How to use API enhancements

[Fullscreen Companion Window](https://chromestatus.com/feature/5173162437246976) permits sites with the window-management permission to [initiate a multi-screen experience](https://github.com/w3c/window-management/blob/main/EXPLAINER_initiating_multi_screen_experiences.md) from a single user activation. Specifically, this proposed enhancement allows scripts to open a popup window when requesting fullscreen on a specific screen of a multi-screen device. Test this by invoking "Fullscreen slide and open notes" in the [window-placement-demo](https://michaelwasserman.github.io/window-placement-demo).
  - Chrome 104+ enables this enhancement by default.
  - Chrome 102+ supports this enhancement with this flag enabled:
    - `chrome --enable-features=WindowPlacementFullscreenCompanionWindow`

[Fullscreen Capability Delegation](https://chromestatus.com/feature/6441688242323456) allows a frame to relinquish its transient user activation, in order to permit another frame (e.g. child, opener, opened popup, sibling) to request fullscreen. This tangential feature enhances multi-screen web applications by enabling a companion window to offer controls for its fullscreen opener. Specifically, fullscreen capability delegation allows a gesture on a companion window to "swap" the displays used for the fullscreen and companion windows. Test this by invoking "Fullscreen slide and open notes" in the [window-placement-demo](https://michaelwasserman.github.io/window-placement-demo), focusing the companion popup window, and pressing the [s] key to swap screens.
  - Chrome 104+ enables this enhancement by default.
  - Chrome 103+ supports this enhancement with this flag enabled:
    - `chrome --enable-blink-features=CapabilityDelegationFullscreenRequest`

[Screen Labels](https://w3c.github.io/window-management/#api-screendetailed-interface) are intended to be short, user-facing names that help distinguish displays without exposing stable hardware identifiers. Implementations should prefer generic labels such as 'Built-in Display' or 'Display 2', and avoid exposing monitor model names, serial numbers, vendor branding, or other per-device identifiers. The label may be empty when the user agent cannot provide a non-identifying name for the screen.
