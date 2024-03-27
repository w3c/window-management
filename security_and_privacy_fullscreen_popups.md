
# 🟥 Obsolete 🟥

This feature is no longer being pursued as of March 27, 2024. See [explainer](EXPLAINER_fullscreen_popups.md#obsolete).

# Security & Privacy

The following considerations are taken from the [W3C Security and Privacy
Self-Review Questionnaire](https://www.w3.org/TR/security-privacy-questionnaire).

## 2.1. What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?

This feature does not directly expose information to web sites that [`element.requestFullscreen`](https://fullscreen.spec.whatwg.org/#ref-for-dom-element-requestfullscreen%E2%91%A0) doesn't already expose. 

## 2.2. Do features in your specification expose the minimum amount of information necessary to enable their intended uses?

No new information is exposed to web sites.

## 2.3. How do the features in your specification deal with personal information, personally-identifiable information (PII), or information derived from them?

No new information is exposed to web sites or collected from users.

## 2.4. How do the features in your specification deal with sensitive information?

This API does not expose any sensitive information, beyond that described above.

## 2.5. Do the features in your specification introduce new state for an origin that persists across browsing sessions?

No.

## 2.6. Do the features in your specification expose information about the underlying platform to origins?

No information is exposed to web sites and the usage of the feature is gated on `window-management` permission which already exposes data that could be indirectly obtained from this API (e.g. the dimensions of a screen.). See [Window Management Security & Privacy](https://github.com/w3c/window-management/blob/main/security_and_privacy.md).

## 2.7. Does this specification allow an origin to send data to the underlying platform?

No.

## 2.8. Do features in this specification enable access to device sensors?
No.

## 2.9. Do features in this specification enable new script execution/loading mechanisms?

No.

## 2.10. Do features in this specification allow an origin to access other devices?

No.

## 2.11. Do features in this specification allow an origin some measure of control over a user agent’s native UI?

Not directly, but invoking HTML Fullscreen, may [change user agent UI](https://fullscreen.spec.whatwg.org/#ui). This proposal introduces an alternative method for invoking HTML fullscreen on a new window.

## 2.12. What temporary identifiers do the features in this specification create or expose to the web?

None.

## 2.13. How does this specification distinguish between behavior in first-party and third-party contexts?

This feature requires the `fullscreen` [permission policy](https://w3c.github.io/webappsec-permissions-policy/#permissions-policy-http-header-field) in both the opener and the opened content (which will enter fullscreen). Additionally, the opener must be allowed the `window-management` permission policy.

## 2.14. How do the features in this specification work in the context of a browser’s Private Browsing or Incognito mode?

The behavior should be the same as for regular mode, except that the user agent should not persist permission data and should request permission every session.

## 2.15. Does this specification have both "Security Considerations" and "Privacy Considerations" sections?

Yes.

## 2.16. Do features in your specification enable origins to downgrade default security protections?

No.

## 2.17. How does your feature handle non-"fully active" documents?

This feature builds upon the existing [`window.open()`](https://html.spec.whatwg.org/multipage/nav-history-apis.html#dom-open-dev) function and inherits `window.open()`'s user activation requirements for initiating the request. The algorithm ultimately invokes [`element.requestFullscreen`](https://fullscreen.spec.whatwg.org/#ref-for-dom-element-requestfullscreen%E2%91%A0) on the [document element](https://dom.spec.whatwg.org/#ref-for-dom-document-documentelement%E2%91%A0) once it is available but skips checking user activation requirements since it was already checked in this code path by the `window.open()` algorithm.