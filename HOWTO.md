# How to use the Multi-Screen Window Placement API

The Multi-Screen Window Placement API is currently available in Chrome 93 and later behind a flag.

1) Download Chrome [Desktop](https://www.google.com/chrome/).
2) Navigate to `chrome://flags` and enable `Experimental Web Platform features`.
3) Navigate to a test page, e.g. https://michaelwasserman.github.io/window-placement-demo
4) Interact with the test page, which requests permission to use info about your screens to open and place windows.

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
