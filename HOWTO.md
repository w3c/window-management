# How to use the Multi-Screen Window Placement API

The Multi-Screen Window Placement API is currently available in Chrome:
- Chrome 100+ enables Multi-Screen Window Placement APIs by default.
- Chrome 93+ supports Multi-Screen Window Placement APIs with either one of these flags enabled:
  - chrome://flags#enable-experimental-web-platform-features
  - `chrome --enable-blink-features=WindowPlacement`

Try these basic API demos, which request permission to use info about your screens to open and place windows:
- https://michaelwasserman.github.io/window-placement-demo
- https://web.dev/multi-screen-window-placement has a demo: https://window-placement.glitch.me

Here is an example of how to use the API:

```javascript
async function main() {
  // Run feature detection.
  if (!('getScreenDetails' in window)) {
    console.log('The Multi-Screen Window Placement API is not supported.');
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

[Fullscreen Companion Window](https://chromestatus.com/feature/5173162437246976) permits sites with the window-placement permission to [initiate a multi-screen experience](https://github.com/w3c/window-placement/blob/main/EXPLAINER_initiating_multi_screen_experiences.md) from a single user activation. Specifically, this proposed enhancement allows scripts to open a popup window when requesting fullscreen on a specific screen of a multi-screen device. Test this by invoking "Fullscreen slide and open notes" in the [window-placement-demo]( https://michaelwasserman.github.io/window-placement-demo).
  - Chrome 102+ supports this enhancement with this flags enabled:
    - `chrome --enable-features=WindowPlacementFullscreenCompanionWindow`
