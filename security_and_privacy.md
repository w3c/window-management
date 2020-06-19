# Security & Privacy

The following considerations are taken from the [W3C Security and Privacy
Self-Review Questionnaire](https://www.w3.org/TR/security-privacy-questionnaire).

Please also refer to the
[Screen Enumeration questionnaire](https://github.com/webscreens/screen-enumeration/blob/master/security_and_privacy.md),
as this API depends on information exposed there.

## 2.1 What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?

This API does not expose any information directly; it generally consumes info
about the device's display configuration, which would be exposed by the proposed
[Screen Enumeration API](https://github.com/webscreens/screen-enumeration).

That information is used to extend existing interfaces for multi-screen support:
* Element.[`requestFullscreen()`](https://fullscreen.spec.whatwg.org/#dom-element-requestfullscreen) is extended to support a target `Screen`.
* Window.[`open()`](https://html.spec.whatwg.org/multipage/window-object.html#dom-open), specifically with feature strings for `left`, `top`, `width`, and `height`
* Window.[`moveTo()`](https://drafts.csswg.org/cssom-view/#dom-window-moveto) and [`moveBy()`](https://drafts.csswg.org/cssom-view/#dom-window-moveby)
* And Window.[`resizeTo()`](https://drafts.csswg.org/cssom-view/#dom-window-resizeto) and [`resizeBy()`](https://drafts.csswg.org/cssom-view/#dom-window-resizeby) (to a lesser degree)

I will include this in the Explainer, which is still undergoing active refinement. The comment about abuse and inconsistency, which may motivate explorations of deprecation, is specifically referring to the Window.open() feature string bounds, and window.[move|resize][To|By]\(\).
Thanks for the question! Hope that helps!


* [`Element.requestFullscreen()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/requestFullScreen)
is extended to support a target `Screen`.
* Window.[`open()`](https://html.spec.whatwg.org/multipage/window-object.html#dom-open),
[`moveTo()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/moveTo),
and [`moveBy()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/moveBy)
are extended to support coordinates that place the window on any connected
screen, rather than on the current screen of the window or opener, which is a
limitation of some implementers. This may require spec changes to support
multi-screen coordinate systems, which are already implemented by some browsers.

Some existing APIs expose information about the window's placement, for example:
[`Window.screenX`](https://drafts.csswg.org/cssom-view/#dom-window-screenx),
[`screenY`](https://drafts.csswg.org/cssom-view/#dom-window-screeny),
[`outerWidth`](https://drafts.csswg.org/cssom-view/#dom-window-outerwidth), and
[`outerHeight`](https://drafts.csswg.org/cssom-view/#dom-window-outerwidth).
These should be accurate and consistent across browser implementations, which
may require updating or refining spec language, and that may possibly change the
values explosed by some implementers. In particular, `screenX` and `screenY` are
specified relative to the
[`web exposed screen area`](https://drafts.csswg.org/cssom-view/#web-exposed-screen-area),
which refers to a singular output device, and should be updated to clarify the
behavior in multi-screen environments.

If `Screen`, accessed via `Window.screen` does not expose non-standard
[`left`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/left) and
[`top`](https://developer.mozilla.org/en-US/docs/Web/API/Screen/top) values,
then specifying `screenX` and `screenY` relative to the primary screen (rather
than the current screen) may expose some additional information about the
display configuration. In particular, some information about the relative 
placement of multiple screens could be deduced or inferred if windows were
placed on secondary screens, since the window coordinates relative to the
primary screen may exceed the available width of the current screen.
the availWidth  the concerned by determining the 
may be deduced when windows are placed on those screens.

## 2.2 Is this specification exposing the minimum amount of information necessary to power the feature?

Yes.

## 2.3 How does this specification deal with personal information or personally-identifiable information or information derived thereof?

This API does not currently expose any information.

## 2.4 How does this specification deal with sensitive information?

This API does not currently expose any information.

## 2.5 Does this specification introduce new state for an origin that persists across browsing sessions?

No.

## 2.6 What information from the underlying platform, e.g. configuration data, is exposed by this specification to an origin?

None; window bounds are already exposed and could be used to infer screen info.

## 2.7 Does this specification allow an origin access to sensors on a user’s device?

No.

## 2.8 What data does this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.

Some info is already exposed synchronously in the document/frame execution
context via standardized and unstandardized properties of the
[`Window`](https://developer.mozilla.org/en-US/docs/Web/API/Window) interface:
* bounds (in the entire screen space), i.e.
[`screenLeft`](https://developer.mozilla.org/en-US/docs/Web/API/Window/screenLeft),
[`screenTop`](https://developer.mozilla.org/en-US/docs/Web/API/Window/screenTop),
[`innerWidth`](https://developer.mozilla.org/en-US/docs/Web/API/Window/innerWidth),
[`outerWidth`](https://developer.mozilla.org/en-US/docs/Web/API/Window/outerWidth),
[`innerHeight`](https://developer.mozilla.org/en-US/docs/Web/API/Window/innerHeight),
[`outerHeight`](https://developer.mozilla.org/en-US/docs/Web/API/Window/outerHeight)
* scaling factor (unstandardized), i.e.
[`devicePixelRatio`](https://developer.mozilla.org/en-US/docs/Web/API/Window/devicePixelRatio)

Future work around this proposal might expose other window properties e.g.:
* The window state: (e.g. maximized, normal/restored, minimized, snapped)
* The window type (e.g. normal/tab, popup, application)
* Events on changes: (e.g. onmove, onwindowdrag, onwindowdrop, onwindowstate)
* Enumerating the list of existing windows opened for a given worker/origin

## 2.9 Does this specification enable new script execution/loading mechanisms?

No.

## 2.10 Does this specification allow an origin to access other devices?

No.

## 2.11 Does this specification allow an origin some measure of control over a user agent’s native UI?

Not directly. The current proposal aims to enable placement of windows on
displays that do not contain a window accessible to that origin at the time.
Future work might aim to let sites express a preferred window type/style,
but nothing would directly control the native UI.

## 2.12 What temporary identifiers might this specification create or expose to the web?

None.

## 2.13 How does this specification distinguish between behavior in first-party and third-party contexts?

Only first-party contexts should be able to control window placement. Third
party contexts in iframes can already discern information about the dimensions
of their frame; nothing in this proposal should affect that.

## 2.14 How does this specification work in the context of a user agent’s Private \ Browsing or "incognito" mode?

The behavior should be the same as for regular mode.

## 2.15 Does this specification have a "Security Considerations" and "Privacy Considerations" section?

Yes.

## 2.16 Does this specification allow downgrading default security characteristics?

No.