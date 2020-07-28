# Security & Privacy

The following considerations are taken from the [W3C Security and Privacy
Self-Review Questionnaire](https://www.w3.org/TR/security-privacy-questionnaire).

## 2.1 What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?

This feature exposes information about screens connected to the device, which is
necessary for cross-screen window placement use cases explored in the explainer.

Currently, some limited information about the screen that each window currently
occupies is available through its Screen interface and related media queries.
Information about the Window's placement on the current screen (or in more rich
cross-screen coordinates) is already available through the Window interface.

This feature exposes information about all connected screens, additional display
properties of each screen, it surfaces events when the set of screens or their
properties change, and it exposes window placements as multi-screen coordinates.
The newly exposed information has been limited to the minimum required for the
most essential cross-screen window placement features. See the ScreenInfo
definition for the new screen properties exposed and their respective utility.

All newly exposed information could be gated by the proposed `window-placement`
permission, limited to secure contexts, and perhaps limited to top-level frames,
but the specific behavior is left to individual implementers.

It should be noted that exisiting `Window.screenLeft`, and `screenTop`, and
non-standard `Screen.left` and `top` values are generally given in cross-screen
coordinates, so it may already be possible for sites to infer the presence of,
and limited information about, screens other than the one its window currently
occupies. Also, some user agent implementations already allow cross-screen
window placement without permission, making it possible to gather multi-screen
information through brute-force movement of a window outside the current
screen's bounds, checking the resulting position and `window.screen` object.

If `Screen`, accessed via `Window.screen` does not expose non-standard
[`left`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/left) and
[`top`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/top) values,
then specifying `Window.screenX` and `screenY` in multi-screen coordinates
(rather than relative to the current screen) may allow sites to infer the
presence of, and limited information about, screens other than the one its
window currently occupies.

## 2.2 Is this specification exposing the minimum amount of information necessary to power the feature?

Generally, yes. The information exposed is widely useful to a variety of window
placement use cases, but not all information will be relevant to every use case.
Gating access with a permission gives users control over which sites, if any,
can access this information.

Supporting queries for limited pieces of information is not directly useful to
sites conducting window placement use cases, and does not specifically prohibit
abusive sites from requesting all available information. Limiting the overall
information that could be exposed with permission would render the API useless.

For example, user agents could show a UI to select the screen used for element
fullscreen, but this has no real advantage for users over the cumbersome current
pattern of dragging windows between screens before entering fullscreen.
Additionally, user agents lack the context around a web application's window
placement actions, and declarative arrangements specified by a site are unlikely
to provide the requisite expresiveness for every use case.

## 2.3 How does this specification deal with personal information or personally-identifiable information or information derived thereof?

Information about the device's screens and the placement of a site's windows are
exposed to the site when the user grants permission, and when other conditions
are satisfied (e.g. secure context).

## 2.4 How does this specification deal with sensitive information?

This API does not expose any sensitive information, beyond that described above.

## 2.5 Does this specification introduce new state for an origin that persists across browsing sessions?

The user agent could persist screen permission grants.

## 2.6 What information from the underlying platform, e.g. configuration data, is exposed by this specification to an origin?

This API proposes exposing about 9-17 new properties for each connected screen,
most of which directly correlate with underyling platform configuration data.
See ScreenInfo's definition for properties exposed and their respective utility.

## 2.7 Does this specification allow an origin access to sensors on a user’s device?

No. If anything, this API may expose the presence of touch sensors associated
with each display, but not access to sensor data itself.

## 2.8 What data does this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.

The API exposes a set of `ScreenInfo` objects, providing a snapshot of info
about each connected display, similar in shape to the interface of the singlular
[`Screen`](https://developer.mozilla.org/en-US/docs/Web/API/Screen) object
currently available to each window, and similar to the set of `Screen` objects
available to an origin if separate windows for that origin were placed on each
of the connected screens, by aggregating the respective `window.screen` objects.

This API currently proposes exposing these properties on each ScreenInfo object:
* The width of the available screen area
  * Standardized as [Screen.availWidth](https://drafts.csswg.org/cssom-view/#dom-screen-availwidth)
* The height of the available screen area
  * Standardized as [Screen.availHeight](https://drafts.csswg.org/cssom-view/#dom-screen-availheight)
* The width of the screen area
  * Standardized as [Screen.width](https://drafts.csswg.org/cssom-view/#dom-screen-width)
* The height of the screen area
  * Standardized as [Screen.height](https://drafts.csswg.org/cssom-view/#dom-screen-height)
* The bits allocated to colors for a pixel of the screen
  * Standardized as [Screen.colorDepth](https://drafts.csswg.org/cssom-view/#dom-screen-colordepth)
* The bits allocated to colors for a pixel of the screen
  * Standardized as [Screen.pixelDepth](https://drafts.csswg.org/cssom-view/#dom-screen-pixeldepth)
* The orientation type of the screen
  * Standardized as [Screen.orientation.type](https://w3c.github.io/screen-orientation/#dom-screenorientation-type)
* The orientation angle of the screen
  * Standardized as [Screen.orientation.angle](https://w3c.github.io/screen-orientation/#dom-screenorientation-angle)
* The left screen coordinate
  * Not standardized, but already exposed by some browsers via
    [`Screen.left`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/left)
* The top screen coordinate
  * Not standardized, but already exposed by some browsers via
    [`Screen.top`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/top)
* The left available coordinate
  * Not standardized, but already exposed by some browsers via
    [`Screen.availLeft`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/availLeft)
* The top available coordinate
  * Not standardized, but already exposed by some browsers via
    [`Screen.availTop`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/availTop)
* Whether it is internal (built-in) or external
  * Not web-exposed, but available via the Chrome Apps
    [`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo),
* Whether it is the primary display or a secondary display
  * Not web-exposed, but available via the Chrome Apps
    [`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo)
* Display scale factor
  * Not standardized, but already exposed by some browsers via
    [`Window.devicePixelRatio`](https://developer.mozilla.org/en-US/docs/Web/API/Window/devicePixelRatio)
* An identifier for the screen
  * Not web-exposed, but a more persistent id is available via the Chrome Apps
    [`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo)
* Whether the screen supports touch input
  * Not web-exposed, but available via the Chrome Apps
    [`system.display` API](https://developer.chrome.com/apps/system_display#method-getInfo)

Some window placement information is already exposed by the
[`Window`](https://drafts.csswg.org/cssom-view/#extensions-to-the-window-interface)
interface, but the proposal requires that bounds be specified in a cross-screen
coordinate space, not relative to the window's current screen:
* Bounds (in the web-exposed screen space), i.e.
[`screenLeft`](https://developer.mozilla.org/en-US/docs/Web/API/Window/screenLeft),
[`screenTop`](https://developer.mozilla.org/en-US/docs/Web/API/Window/screenTop),
[`innerWidth`](https://developer.mozilla.org/en-US/docs/Web/API/Window/innerWidth),
[`outerWidth`](https://developer.mozilla.org/en-US/docs/Web/API/Window/outerWidth),
[`innerHeight`](https://developer.mozilla.org/en-US/docs/Web/API/Window/innerHeight),
[`outerHeight`](https://developer.mozilla.org/en-US/docs/Web/API/Window/outerHeight)
* scaling factor (unstandardized), i.e.
[`devicePixelRatio`](https://developer.mozilla.org/en-US/docs/Web/API/Window/devicePixelRatio)

## 2.9 Does this specification enable new script execution/loading mechanisms?

No.

## 2.10 Does this specification allow an origin to access other devices?

This API allows an origin to read properties of screens connected to the device.
The screen may be internally connected (e.g. the built-in display of a laptop)
or externally connected (e.g. an external monitor).

An origin cannot use this API to send commands to the displays, so hardening
against malicious input is not a concern.

Enumerating the screens connected to the device does provide significant
entropy. If multiple computers are connected to the same set of screens, an
attacker may use the display information to deduce that those computers are in
the same physical vicinity. To mitigate this issue, user permission is required
to access the display list, the API is only available on secure contexts, and
the ids of each screen are generated on a per-user per-origin basis, reset when
cookies are cleared.

## 2.11 Does this specification allow an origin some measure of control over a user agent’s native UI?

No.

## 2.12 What temporary identifiers might this specification create or expose to the web?

None.

## 2.13 How does this specification distinguish between behavior in first-party and third-party contexts?

Only first-party contexts can generally control the window's placement. Third
party contexts in iframes can already discern the dimensions of their frame;
nothing in this proposal should affect that. Additionally, third party contexts
can request element fullscreen (with the `allowfullscreen` feature policy) and
open and place popup windows; this proposal extends those functionalities for
cross-screen requests, with similar permission requirements to the first-party
context of the main frame.

## 2.14 How does this specification work in the context of a user agent’s Private \ Browsing or "incognito" mode?

The behavior should be the same as for regular mode, except that the user agent
should not persist permission data and should request permission every session.

## 2.15 Does this specification have a "Security Considerations" and "Privacy Considerations" section?

Yes.

## 2.16 Does this specification allow downgrading default security characteristics?

No.

## 2.17. What should this questionnaire have asked?

The questionnaire could ask if implementing the proposal would yield or enable
any additional Security and Privacy protections.

By adding the proposed `window-placement` permission, browsers could further
limit unpermissioned access to existing information and capabilities.

For example, without the permission, browsers could:
* Disregard `left` and `top` values requested via window.open()'s
  [features](https://developer.mozilla.org/en-US/docs/Web/API/Window/open#Window_features)
  argument string to reduce clickjacking risks.
* Disregard requests to move popup and web application windows via
  [`window.moveTo(x, y)`](https://developer.mozilla.org/en-US/docs/Web/API/Window/moveTo)
  to reduce clickjacking risks.
* Return the corresponding innerWidth and innerHeight values for Window's
  [`outerWidth`](https://drafts.csswg.org/cssom-view/#dom-window-outerwidth)
  and
  [`outerHeight`](https://drafts.csswg.org/cssom-view/#dom-window-outerheight)
  to limit information exposed about the window frame size, which can be used to
  infer more granular information about the user agent, its version, and the
  window frame type used than would otherwise be exposed.
* Return 0 for Window's
  [`screenX`](https://drafts.csswg.org/cssom-view/#dom-window-screenx) and
  [`screenY`](https://drafts.csswg.org/cssom-view/#dom-window-screeny)
  attributes to limit similar unnecessary information exposure.
* Perhaps limit other existing Screen and Window information and capabilities.

The questionnaire could also dig deeper into potential security concerns of the
API, which are explored in the Privacy & Security section of the explainer.