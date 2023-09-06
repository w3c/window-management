# Window Management on the Web - Creating Fullscreen Popup Windows

## Introduction
This proposal aims to reduce user and developer friction for initiating a fullscreen experience on the desired display of a multi-screen device. The proposed web platform enhancement allows web applications to open a new window in HTML [fullscreen](https://fullscreen.spec.whatwg.org/) mode on a specific display with a single user gesture.

## Background
[Window Management](https://github.com/w3c/window-management) is a new permission-gated web platform API that provides web applications with information about the device's connected screens, allowing them to open and position windows or request fullscreen on any of those screens.

The HTML living standard specifies [Tracking User Activation](https://html.spec.whatwg.org/multipage/interaction.html#tracking-user-activation), to gate some web platform APIs on user interaction signals. 

## Problem
Web applications are limited in the ways that they can [Initiate Multi-Screen Experiences](https://github.com/w3c/window-management/blob/main/EXPLAINER_initiating_multi_screen_experiences.md) across multi-display configurations. Specifically, opening a popup and transitioning the window to fullscreen typically requires two separate user gestures (see [Spec Notes](#spec-notes)). This results in a cumbersome user experience for web applications which utilize multiple displays. Additionally, web apps wish to show fullscreen content on a specific display, alongside the original content, but target-display fullscreen may make the existing window inaccessible

Multiple partners and early adopters of the [Window Management API](https://w3c.github.io/window-management/) have specifically requested the ability to show one or more fullscreen popups from a single user gesture, to avoid the user friction entailed with making separate gestures to create each window and make each window fullscreen. For example: 
*   [window.open should support the 'fullscreen' option](https://github.com/w3c/window-management/issues/7)
*   [Feature Request: Initiate a multi-screen experience from a single user activation](https://github.com/w3c/window-management/issues/98#top)
*   [Feature request: Fullscreen support on multiple screens](https://github.com/w3c/window-management/issues/92).

## Goals
The goal of this document is to outline the necessary spec changes that would permit web applications to open a new fullscreen window on a specific display, with only a single user gesture. It also aims to explore abuse and other security / privacy considerations. In particular this document focuses on introducing straightforward API surface and corresponding algorithmic changes to allow such behavior.

The [Initiating Multi-Screen Experiences Explainer](https://github.com/w3c/window-management/blob/main/EXPLAINER_initiating_multi_screen_experiences.md) covers some related goals (i.e. launching N fullscreen windows on N displays) not covered in this proposal.

## Use Cases
Some basic illustrative examples which may be achieved with this proposal are listed below:
*   Video streaming app launches a video directly to fullscreen on a separate display.
*   3D modeling app launches a preview in fullscreen on a separate display.
*   Medical report app opens imagery fullscreen on a specialized display.
*   Point-of-sale app opens a fullscreen payment window on a customer display.

This proposal is limited to launching at most one fullscreen popup onto one display, given a single transient user activation. [Future enhancements](https://github.com/w3c/window-management/blob/main/EXPLAINER_initiating_multi_screen_experiences.md) could allow a web application to launch multiple fullscreen popup windows from a single transient user activation, to allow for more use cases:

*   Financial app opens a fullscreen dashboard on the primary monitor and a fullscreen stock tracker window on a secondary display.
*   Virtual desktop app launches fullscreen windows on primary and secondary displays to mimic a remote monitor configuration.
*   Security monitoring app launches 6 fullscreen video feeds on an array of 6 displays.
*   Gaming app launches 3 fullscreen windows across 3 displays to provide a continuous widescreen view.

## Proposal
Allow sites with the `window-management` permission to open a fullscreen window on a specified display from a single user gesture. Scripts calling  <code>[window.open()](https://html.spec.whatwg.org/multipage/window-object.html#dom-open-dev)</code> to request a popup window could also include a new <code>fullscreen</code> boolean window feature.

Fullscreen popup requests could also include window bounds features (e.g. `left`, `top`, `width`, `height`), with positions specified [relative to the multi-screen origin](https://w3c.github.io/window-management/#api-window-attribute-and-method-definition-changes), to request that the window be made fullscreen on the screen containing those bounds, and also to be used as the popup window bounds after exiting fullscreen.

### Example Code

#### Basic Example:
```js
window.open(url, '_blank', 'popup,fullscreen');
```

#### Opening a fullscreen popup on another display:
```js
const otherScreen = screenDetails.screens.find(s => s !== screenDetails.currentScreen);
window.open(url, '_blank', `left=${otherScreen.availLeft},` +
                            `top=${otherScreen.availTop},` +
                            `width=${otherScreen.availWidth},` +
                            `height=${otherScreen.availHeight},` + 
                            `fullscreen`);
```

### Spec Changes
The spec does not contain an exhaustive list of valid features in the `features` argument, however, the spec does contain procedures for determining if a [popup window is requested](https://html.spec.whatwg.org/multipage/window-object.html#popup-window-is-requested), and the specific `tokenizedFeatures` to look for. Following this model, the spec shall be updated to define a similar algorithm for checking if a **fullscreen window** is requested. A fullscreen window must also first be considered a popup window.

After [checking if a popup window is requested](https://html.spec.whatwg.org/multipage/window-object.html#popup-window-is-requested), add the following section:

To **check if a fullscreen window is requested**, given _tokenizedFeatures_:
1. If _tokenizedFeatures_ is empty, then return false.
2. Let _popup_ be the result of [checking if a popup window is requested](https://html.spec.whatwg.org/multipage/window-object.html#popup-window-is-requested).
3. If _popup_ is false, then return false.
4. Let _fullscreen_ be the result of [checking if a window feature is set](https://html.spec.whatwg.org/multipage/window-object.html#window-feature-is-set), given _tokenizedFeatures_, "`fullscreen`", and false.
5. If _fullscreen_ is false, then return false.
6. Let _permissionPolicyWindowManagementState_ be the result of [Is feature enabled in document for origin?](https://www.w3.org/TR/permissions-policy/#is-feature-enabled) given feature "`window-management`", the current [Document](https://dom.spec.whatwg.org/#document) and the current [origin](https://url.spec.whatwg.org/#concept-url-origin).
7. If _permissionPolicyWindowManagementState_ is "Disabled", then return false.
8. Let _permissionPolicyFullscreenState_ be the result of [Is feature enabled in document for origin?](https://www.w3.org/TR/permissions-policy/#is-feature-enabled) given "`fullscreen`", the current [Document](https://dom.spec.whatwg.org/#document) and the current [origin](https://url.spec.whatwg.org/#concept-url-origin).
9. If _permissionPolicyFullscreenState_ is "Disabled", then return false.
10. If the [transient activation state](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation) is false, then return false.
11. Let _permissionState_ be the result of [getting the current permission state](https://w3c.github.io/permissions/#dfn-getting-the-current-permission-state) given "`window-management`".
12. If _permissionState_ is not "[granted](https://w3c.github.io/permissions/#dom-permissionstate-granted)", then return false.
13. [Consume user activation](https://html.spec.whatwg.org/multipage/interaction.html#consume-user-activation).
14. Return true.

Add a Note below this algorithm which reads:
> User agents are encouraged to restore the user-agent-adjusted window.open() specified bounds, or their inferred defaults when a fullscreen popup exits fullscreen.

In [7.3.2 Browsing contexts](https://html.spec.whatwg.org/multipage/document-sequences.html#windows), insert a new list item after the [is popup](https://html.spec.whatwg.org/multipage/document-sequences.html#is-popup) item and its corresponding note. The new item should read: "An **is fullscreen** boolean, initially false." with an anchor of #is-fullscreen.

Add a note below the item which reads:
> If **is fullscreen** is true, user agents should [requestFullscreen](https://fullscreen.spec.whatwg.org/#dom-element-requestfullscreen) on the [browsing context’s](https://html.spec.whatwg.org/multipage/document-sequences.html#browsing-context) active document [documentElement](https://dom.spec.whatwg.org/#dom-document-documentelement) once the `documentElement` is ready.

After step 12.1 in **[window open steps](https://html.spec.whatwg.org/multipage/nav-history-apis.html#window-open-steps)**, after "_Set targetNavigable's active browsing context's is popup to the result of [...]_", insert a step:
 > Set targetNavigable's [active browsing context](https://html.spec.whatwg.org/multipage/document-sequences.html#nav-bc)'s is fullscreen to the result of [checking if a fullscreen window is requested](https://html.spec.whatwg.org/multipage/nav-history-apis.html#popup-window-is-requested), given tokenizedFeatures.

"is fullscreen" should link to the list item anchor (#is-fullscreen) added to [7.3.2 Browsing contexts](https://html.spec.whatwg.org/multipage/document-sequences.html#windows) as previously described.


#### Spec Notes
**[`window.open()`](https://html.spec.whatwg.org/multipage/window-object.html#dom-open-dev) Activation Consumption**

The published specifications do not specify that the [`window.open()`](https://html.spec.whatwg.org/multipage/window-object.html#dom-open-dev) algorithm consumes the [transient activation](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation), nor does any spec list it as an [activation consuming API](https://html.spec.whatwg.org/multipage/interaction.html#activation-consuming-api). In the [`window.open()`](https://html.spec.whatwg.org/multipage/window-object.html#dom-open-dev) [window open steps](https://html.spec.whatwg.org/multipage/nav-history-apis.html#window-open-steps), the [rules for choosing a navigable](https://html.spec.whatwg.org/multipage/document-sequences.html#the-rules-for-choosing-a-navigable) describe procedures for checking if a [transient activation](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation) exists, but nothing describing that it should consume it, presumably leaving it up to the user agent to decide based on their own security and popup blocking measures.

However, in practice, multiple user agents (Chrome, Firefox, Safari) <em>do</em> consume the [transient activation](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation) when popup blocking is enabled. Additionally, some conflicting documentation has been published which describes [`window.open()`](https://html.spec.whatwg.org/multipage/window-object.html#dom-open-dev) as an activation consuming API:
*   [MDN Docs](https://developer.mozilla.org/en-US/docs/Web/Security/User_activation#transient_activation) describe [`window.open()`](https://html.spec.whatwg.org/multipage/window-object.html#dom-open-dev) as an example API which consumes the user activation.
*   A Chrome developer [blog post](https://developer.chrome.com/blog/user-activation/#how-does-user-activation-v2-work) describes [`window.open()`](https://html.spec.whatwg.org/multipage/window-object.html#dom-open-dev) as an example API which consumes the user activation.

Published specifications may need some corrective action, but that is outside the scope of this proposal. The proposal in this document makes the assumption that user agents <strong>may or may not</strong> consume the transient activation in [`window.open()`](https://html.spec.whatwg.org/multipage/window-object.html#dom-open-dev).

### Considerations

#### **Overlapping Fullscreen Windows**
This proposal does not define behavior for certain scenarios which should be considered by user agents implementing the proposal:
*   Opening a fullscreen popup on a display which already contains a fullscreen window.
*   Opening a fullscreen popup on the same display as the opener, which may or not be fullscreen.
*   Pre-existing protections prevent placing popups over fullscreen
    *   Chrome exits fullscreen (see [ForSecurityDropFullscreen](https://source.chromium.org/chromium/chromium/src/+/main:content/public/browser/web_contents.h?q=ForSecurityDropFullscreen))
    *   Firefox exits fullscreen
    *   Safari opens the popup in a separate fullscreen space

#### **Feature detection**
There is no standard API for querying which features are supported in the [`window.open()`](https://html.spec.whatwg.org/multipage/window-object.html#dom-open-dev) `features` argument. There is no way for a web application to check that the user agent supports the `fullscreen` key in the `features` argument without calling the API and detecting if the resulting popup window's [`Document.fullscreenElement`](https://fullscreen.spec.whatwg.org/#ref-for-dom-document-fullscreenelement%E2%91%A0) is non-null.

The ability to detect supported keys of the `features` argument is out of scope for this proposal. Future [improvements](https://github.com/whatwg/html/issues/7485) to `window.open()` could allow for feature detection.

#### **Requiring popup initial boundaries**
Scripts may specify window boundaries through the `left`, `top`, `width`, `height` features of [`window.open()`](https://html.spec.whatwg.org/multipage/window-object.html#dom-open-dev) which, under this proposal, control which display the window will appear fullscreen on. Leaving out these parameters may cause undefined behavior (e.g. appearing on the display which contains screen coordinates (0,0)). As a more concrete solution, the algorithmic check for fullscreen could require that `left` and `top` are set which always require the web application to specify a display, and leaves less room for decision by the user agent. However, since `left`, `top`, `width`, `height` are already not required for popup windows, it's reasonable to let the user agent use the existing defaults and fullscreen the window on whichever display would have contained the popup if the `fullscreen` parameter wasn't specified.

#### **Other fullscreen windows**
This proposal requires that a new fullscreen window must first be also considered a popup window. Future enhancements may consider allowing other types of windows to be opened in fullscreen.

#### **Fullscreen Exit Behavior**
The proposed note specifies that user agents are encouraged to restore the window to specified bounds after fullscreen exits. Some alternative examples include: Closing the window, restoring to a popup with bounds expanded to each edge of the display.

#### **Asynchronous behavior**
`Element.requestFullscreen()` returns a promise that resolves after the element becomes fullscreen, but Window.open() does not return a promise. Web applications would need alternative methods of detecting when the window is open, ready and fullscreen.

#### **fullscreenchange event**
The browser may omit a `fullscreenchange` event prior to any scripts being executed in the new popup window frame. Web applications expecting a `fullscreenchange` would need to account for this behavior in fullscreen-popup scenarios, or browsers would need to carefully ensure that `fullscreenchange` is not emitted until after scripts are ready to handle the event.

#### **Delays entering fullscreen**
There may exist a delay (unintentional or malicious) between the time the popup is created during the original [user activation](https://html.spec.whatwg.org/multipage/interaction.html#tracking-user-activation) and the time that the [document element](https://dom.spec.whatwg.org/#document-element) becomes available. Slow network conditions or intentional throttling by a malicious server may introduce a significant delay between the popup window opening and the popup's document element becoming available to fullscreen. In such case, the user agent could postpone or abort the steps for making the popup's document element fullscreen. 

#### **Cross-Origin Popups**
The proposal allows fullscreen popup requests for cross-origin content. If Site A is granted `window-management` and requests a fullscreen popup for Site B, then the browser shall allow Site B to open fullscreen. Site B may opt-out of fullscreen by using the appropriate [permission policy headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy) and even set them dynamically based on the [Referer header](https://www.rfc-editor.org/rfc/rfc9110#field.referer) which contains the URL which opened the popup. There is also a risk for Site A to open a malicious Site B. However, there is an implicit chain of trust via the `window-management` permission suggesting that the user trusts Site A to not open malicious content (regardless of origin). This should be compared to traditional HTML fullscreen of cross-origin iframes, and opening non-fullscreen cross-origin popups. 

#### **HTTP Redirects**
HTTP redirects are supported. When Site A opens a fullscreen popup for Site B, but Site B provides an [HTTP Redirect](https://developer.mozilla.org/en-US/docs/Web/HTTP/Redirections) to a different origin it will be treated as a cross-origin popup. See [Cross-Origin Popups](#cross-origin-popups).

#### **Fullscreen hidden content**
Fullscreen popups allow sites to fullscreen content which was not previously visible to users. However, there are already ways to achieve this with [Element.requestFullscreen](https://fullscreen.spec.whatwg.org/#ref-for-dom-element-requestfullscreen%E2%91%A0) (e.g. unhide an element immediately before requesting fullscreen).

## Security Considerations
A notable security consideration stems from the fact that the web application may launch a fullscreen window on a display that the user is not looking at, since the [user activation](https://html.spec.whatwg.org/multipage/interaction.html#tracking-user-activation) (i.e. button click) may have occurred on another display which the user is focused on. The user may not notice the fullscreen window transition, nor the fullscreen bubble (e.g. Firefox's "&lt;origin> is now full screen [Exit Full Screen (Esc)]" or Chrome's "Press [Esc] to exit full screen") which could allow for a malicious application to mimic other applications or the operating system without the user realizing that it is a browser window.

This is partially mitigated by gating this feature on the "`window-management`" permission. Users should only grant permissions for [powerful features](https://w3c.github.io/permissions/#dfn-powerful-feature) to web applications that they trust.

This could also be mitigated further by having the user agent show a fullscreen bubble when the user first interacts with the cross-display fullscreen window or by showing a bubble on the screen where the user activation took place (i.e. "Window went fullscreen on display ###"), or both. In these scenarios, there is a higher confidence that the user will see a fullscreen bubble.

## Privacy Considerations
This feature does not expose any information to sites, and there are no privacy considerations to note beyond those already documented in the Window Management [Privacy Considerations](https://www.w3.org/TR/window-management/#privacy) section. The feature is gated behind the "`window-management`" permission, so it does not expose any information outside what is already available from the API when permission is granted.

## Alternatives Considered

### Keep the existing behavior.
Web applications can create a popup window that fills the available display bounds, as a poor substitute of HTML fullscreen. However, this contains window borders and decorations not appropriate for some use cases.

### Allow a *target-screen* fullscreen request after opening a *cross-screen* popup.
*   Requires more complex changes to user activation signals, which has several consequences:
    *   Requires the user agent to maintain transient user activation signal(s) on one or more frames (the opener and/or the new window and its descendant frames).
    *   Less straightforward, which could reduce adoption.
    *   Propagating a transient user activation signal to a new popup or otherwise relaxing the user activation requirements for [`element.requestFullscreen()`](https://fullscreen.spec.whatwg.org/#ref-for-dom-element-requestfullscreen%E2%91%A0) in this scenario would likely raise significant additional concerns. 
*   Less ergonomic for application developers.
    *   A web developer would need to manually call [`element.requestFullscreen()`](https://fullscreen.spec.whatwg.org/#ref-for-dom-element-requestfullscreen%E2%91%A0) from within the popup after it has loaded, or call [`element.requestFullscreen()`](https://fullscreen.spec.whatwg.org/#ref-for-dom-element-requestfullscreen%E2%91%A0) from the opener’s script after calling [`window.open()`](https://html.spec.whatwg.org/multipage/window-object.html#dom-open-dev). Both versions would require different user activation signal propagation strategies as mentioned above.

### Allow fullscreen capability delegation to the new window after creating a popup.
Utilize [capability delegation](https://wicg.github.io/capability-delegation/spec.html) to allow the opener window to delegate fullscreen capabilities to the new popup window via` window.postMessage()`. This currently does not work since `window.open()` consumes the transient user activation which prevents capability delegation.

This approach has similar disadvantages to the preceding alternative in that it is less intuitive for developers and requires additional consideration for transient user activation changes. An advantage to this approach is that it gives the developers more control over which element to fullscreen in the resulting windows document, but the proposed solution also allows this with some (albeit less straightforward) procedures.