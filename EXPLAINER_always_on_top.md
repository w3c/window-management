# Explainer: `alwaysOnTop` option for `window.open()`

## 1. Introduction & Abstract

Currently, web applications cannot create standalone, independent windows that stay above other desktop applications. While the `Document Picture-in-Picture API` does allow for always-on-top windows, they are strictly tied to the lifecycle of the initiating tab and close automatically if that tab navigates or closes. This explainer proposes a new boolean option for `window.open()`, called `alwaysOnTop`. When set to `true`, the browser requests that the operating system keep the newly created window pinned above other non-always-on-top windows, enabling fully standalone, always-on-top utility windows that can outlive the tab that opened them.

---

## 2. Use Cases & User Motivation

There are several key productivity and utility use cases where users benefit from persistent, always-on-top windows:

* **Meeting Note-Taking & Reference Companions:** A persistent companion window opened by a dedicated note-taking web app to take notes alongside a separate meeting application (e.g., Google Meet, Zoom, Microsoft Teams) or presentation, ensuring the notepad stays visible above full-screen windows without getting lost when switching focus.
* **Developer Tools & Live Telemetry:** Live-reloading logs, debug dashboards, or performance monitors that developers want visible on-screen while coding inside an IDE or interacting with full-screen terminal sessions.
* **Persistent Utilities & Scratchpads:** Standalone calculators, timers, quick-translators, or reference widgets that users need pinned above various full-screen or tiled desktop apps.

### The User-Facing Problem with Opener-Coupled Windows

Existing web platform capabilities (such as Document Picture-in-Picture) require an always-on-top view to remain strictly bound to the lifecycle of the opener tab. In real-world workflows, this creates distinct user-facing friction:

1. **Inability to "Launch and Close" (The Hostage Opener Tab):**
   When launching a standalone companion window (such as a meeting note-taking companion or persistent scratchpad), users naturally want to close or navigate away from the launching tab or primary browser window to declutter their workspace, reduce visual distraction, and free system memory. With opener-bound windows, closing or navigating the opener tab immediately terminates the always-on-top window. This creates anxiety during critical workflows (e.g., inadvertently closing a companion notes window during a meeting by closing an unrelated browser tab) and forces users to keep a "dead" or unnecessary tab open simply to act as a lifecycle anchor.

2. **Tab Clutter & Hidden Resource Overhead:**
   Users are forced to keep tabs buried across windows solely to preserve the floating window. This not only clutters the tab strip, but also maintains unwanted background connections and heavyweight document memory overhead for the opener tab when the user only intended to run a lightweight companion.

3. **Mental Model Mismatch (Opener vs. Standalone Tool):**
   Users conceptualize a pinned utility as an independent desktop tool rather than a child view of a specific browser tab. Forcing a dual-entity relationship (controller tab + floating view) violates user expectations and complicates window management.

---

## 3. Proposed API

We propose adding `alwaysOnTop` to the `windowFeatures` parameter of `window.open()`, as well as introducing a corresponding read-only boolean `alwaysOnTop` property directly onto the `Window` interface.

```javascript
// Proposed syntax using the traditional comma-separated string
const utilityWindow = window.open(
  'https://example.com/controls', 
  'Controls', 
  'width=300,height=200,alwaysOnTop=true'
);

// Read-only boolean property reflection
console.log(utilityWindow.alwaysOnTop); // true (if successfully opened as always-on-top)
```

### Expected Behavior

* **Permission Requirement:** This capability is gated behind the `window-management` permission. If this permission has not been granted by the user, the `alwaysOnTop` request is silently ignored, and a standard popup window is created instead.
* If `true` and the required permission is granted, the OS-level window manager is instructed to keep this window pinned above other non-always-on-top windows.
* **Z-Ordering of Multiple Windows:** The relative z-ordering between multiple concurrent always-on-top windows is left to the User Agent or the underlying operating system. For example, UAs may order them based on which window was most recently focused.
* **UA-Defined Limits:** To preserve usability, browsers (User Agents) may enforce implementation-specific limits. For example, UAs may restrict origins to a single active `alwaysOnTop` window at a time, or make it mutually exclusive with other types of Picture-in-Picture (PiP) windows. The UA may also restrict the size of the always-on-top window.
* **Popup Window Context Requirement:** The `alwaysOnTop` hint only applies when `window.open()` creates a new, standalone popup window context. If the call results in a new tab within an existing multi-tab browser window, or targets and navigates an existing window (e.g., via a named `target`), the `alwaysOnTop` request is silently ignored.
* If the user manually minimizes the window, it should obey.
* The `window.alwaysOnTop` property is read-only after creation and reflects whether the window is currently pinned above other windows (returning `true`) or running in standard/fallback mode (returning `false`).

---

## 4. Feature & Permission Detection

Exposing the read-only `alwaysOnTop` property on the `Window` interface addresses two critical detection challenges for developers:

1. **Feature Detection:** Developers can synchronously check for browser API support by verifying the existence of `'alwaysOnTop' in Window.prototype`.
2. **State Verification:** A script running inside the new popup (or the opening application holding a reference to it) can confidently determine if the window was legitimately successfully opened with the always-on-top capability by checking the `window.alwaysOnTop` boolean.

```javascript
async function openUtilityWindow() {
  let canUseAlwaysOnTop = false;

  // 1. Feature Detection
  if ('alwaysOnTop' in Window.prototype) {
    // 2. Permission Detection & Request
    try {
      let permissionStatus = await navigator.permissions.query({ name: 'window-management' });

      if (permissionStatus.state === 'prompt') {
        // Explicitly request the permission (requires user gesture)
        await window.getScreenDetails();
        permissionStatus = await navigator.permissions.query({ name: 'window-management' });
      }

      canUseAlwaysOnTop = permissionStatus.state === 'granted';
    } catch (e) {
      // Keep false if query or prompt is not supported/denied
    }
  }

  if (canUseAlwaysOnTop) {
    // Open as an always-on-top window
    const win = window.open('/tool', 'Tool', 'width=300,height=200,alwaysOnTop=true');
    console.log("Is always-on-top:", win.alwaysOnTop); // true
    return;
  }

  // Fallback 1: Document Picture-in-Picture
  if ('documentPictureInPicture' in window) {
    const pipWin = await documentPictureInPicture.requestWindow({
      width: 300,
      height: 200,
    });
    // Load content into the PiP window (e.g., via iframe)
    const iframe = pipWin.document.createElement('iframe');
    iframe.src = '/tool';
    iframe.style.width = '100%';
    iframe.style.height = '100%';
    iframe.style.border = 'none';
    pipWin.document.body.appendChild(iframe);
    return;
  }

  // Fallback 2: Standard Window
  const win = window.open('/tool', 'Tool', 'width=300,height=200');
  console.log("Is always-on-top:", win.alwaysOnTop); // false
}
```

---

## 5. Security & Privacy Considerations

Because an "always on top" window can be abused for disruptive purposes (e.g., spoofing system UI, creating un-dismissible advertisements, phishing, or screen obscuration), strict mitigations are required:

* **Dual Gating (Permission + Popup Blocker / User Activation):**
  * **Permission Gating:** The `alwaysOnTop` capability is gated behind the `window-management` permission. If this permission is not granted, `window.open(..., 'alwaysOnTop=true')` degrades gracefully to a regular popup window without always-on-top elevation.
  * **Transient User Activation & Popup Policy:** Opening an always-on-top window requires transient user activation (e.g., a direct user gesture) and adheres to standard browser "Popups and Redirects" enforcement (`chrome://settings/content/popups`).
  * Note: Certain UA permissions (e.g. `chrome://settings/content/popups`) can grant websites the ability to bypass the user gesture requirement for `window.open()`, which also means that they can bypass the user gesture requirement for always-on-top windows. We don't consider this to be a problem given the additional `window-management` permission requirement.
  * Requiring both the requestable `window-management` permission and explicit user activation constitutes a very high bar that prevents untrusted or malicious sites from spamming pinned windows.
* **Move & Resize Restrictions:** To prevent abuse scenarios like a window programmatically tracking the cursor or expanding to obscure critical OS interface elements, synchronous APIs that manipulate window bounds (`moveTo`, `moveBy`, `resizeTo`, `resizeBy`) are subject to strict UA rate limits and transient user activation checks, analogous to the mitigations applied to Document Picture-in-Picture windows.
* **Focus Stealing Protections:** Always-on-top windows must not be permitted to aggressively force focus back to themselves when the user interacts with other applications.
* **Explicit User Affordance & Easy Dismissal:** The browser and operating system window frames must provide clear, un-spoofable user controls to dismiss the window at any time.

---

## 6. Alternatives Considered

### Document Picture-in-Picture API

The [Document Picture-in-Picture API](https://wicg.github.io/document-picture-in-picture/) enables web developers to open an always-on-top window populated with arbitrary HTML content. While valuable for background media presentation, it is not well-suited for standalone always-on-top utilities for several reasons:

1. **Strict Lifecycle Coupling ("Close-on-Destroy"):**
   Document PiP windows are inherently tied to the lifecycle of the opener page. If the user closes the opener tab, navigates to another page, or closes the primary browser window, the PiP window is destroyed immediately. This makes it impossible to implement workflows where a user launches a utility window and then closes the initiating browser tab to save resources or declutter.
2. **Mental Model & Opener UX Failure:**
   Document PiP requires the user to mentally manage two separate entities: the opener tab ("controller") and the PiP window ("view"). For applications that function as standalone tools or meeting note-taking companions, users expect the window to behave as an independent entity rather than a tethered child view.
3. **Semantic & Architectural Alignment:**
   Document PiP was intentionally specified for transient, secondary views of an active tab while that tab is backgrounded. Stretching PiP to emulate standalone, persistent applications violates its intended design model and adds architectural complexity for both developers and User Agents.

### Native Packaging (Electron / NW.js)

Currently, web developers building tools requiring persistent always-on-top capabilities (such as companion note-taking apps, trading dashboards, or screen annotation widgets) are forced to wrap their applications in desktop runtimes like Electron just to access APIs like `BrowserWindow.setAlwaysOnTop()`. Providing this capability natively on the web platform bridges a critical capabilities gap, eliminating the need to bundle dedicated native runtime binaries.