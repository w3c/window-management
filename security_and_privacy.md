# Security & Privacy

The following considerations are taken from the [W3C Security and Privacy
Self-Review Questionnaire](https://www.w3.org/TR/security-privacy-questionnaire).

Please also refer to the
[Screen Enumeration questionnaire](https://github.com/webscreens/screen-enumeration/blob/master/security_and_privacy.md),
as this API depends on information exposed there.

## 2.1 What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?

This API primarily concerns handling of cross-screen coordinates passed to
existing [`Window`](https://developer.mozilla.org/en-US/docs/Web/API/Window)
interface functions, and extending
[`Element.requestFullscreen()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/requestFullScreen)
to accept a target screen.

Future work may include exposing new window information, controls, and events,
like state (e.g. maximized), and movement (e.g. onmove) etc. This information
would be exposed to offer sites control and introspection for the appearance of
their web content windows.

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