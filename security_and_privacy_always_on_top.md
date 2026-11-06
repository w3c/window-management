# [Self-Review Questionnaire: Security and Privacy](https://w3c.github.io/security-questionnaire/)

> 01. What information does this feature expose, and for what purposes?

This feature enables web applications (with the `window-management` permission) to open standalone always-on-top popup windows via `window.open(..., 'alwaysOnTop=true')` that stay pinned above other non-always-on-top windows independently of the opener tab's lifecycle.

In terms of exposed information, it adds a read-only `window.alwaysOnTopPopup` boolean attribute indicating whether the popup window is currently pinned above other non-always-on-top windows. This is necessary so web applications can detect support for the feature (`'alwaysOnTopPopup' in Window.prototype`) and verify whether a window opened with `alwaysOnTop=true` was granted always-on-top elevation by the user agent and operating system.

> 02. Do features in your specification expose the minimum amount of information necessary to implement the intended functionality?

Yes. Only a single boolean (`window.alwaysOnTopPopup`) is exposed on the `Window` interface.

> 03. Do the features in your specification expose personal information, personally-identifiable information (PII), or information derived from either?

No.

> 04. How do the features in your specification deal with sensitive information?

This feature does not expose or process sensitive information.

> 05. Does data exposed by your specification carry related but distinct information that may not be obvious to users?

No. To avoid leaking cross-origin state (such as whether another origin currently has an always-on-top window open), user agents are encouraged to scope concurrency limits per-origin rather than enforcing a detectable cross-origin cap.

> 06. Do the features in your specification introduce state that persists across browsing sessions?

No new persistent state is introduced beyond the existing `window-management` permission state.

> 07. Do the features in your specification expose information about the underlying platform to origins?

Only indirectly: if the underlying platform or window manager does not support always-on-top windows (such as certain mobile or Wayland environments), `window.alwaysOnTopPopup` will remain `false` even when the `window-management` permission is granted. Usage of the `alwaysOnTop` option is already gated on the `window-management` permission. See [Window Management Security & Privacy](https://github.com/w3c/window-management/blob/main/security_and_privacy.md).

> 08. Does this specification allow an origin to send data to the underlying platform?

No, other than instructing the user agent to request an always-on-top window level from the OS window manager when opening a popup.

> 09. Do features in this specification enable access to device sensors?

No.

> 10. Do features in this specification enable new script execution/loading mechanisms?

No.

> 11. Do features in this specification allow an origin to access other devices?

No.

> 12. Do features in this specification allow an origin some measure of control over a user agent's native UI?

Yes. When a site with the `window-management` permission opens a popup with `alwaysOnTop=true`, the resulting popup window is kept above normal non-always-on-top windows. To mitigate spoofing, phishing, and clickjacking risks:
* The window must always display non-spoofable browser/OS window chrome, including the visible origin and a close button.
* User agents enforce minimum and maximum window size bounds and restrict programmatic repositioning/resizing (`moveTo`, `moveBy`, `resizeTo`, `resizeBy`), analogous to [Document Picture-in-Picture mitigations](https://wicg.github.io/document-picture-in-picture/#spoofing).
* User agents may enforce per-origin concurrency limits and mutual exclusivity with other Picture-in-Picture windows.

> 13. What temporary identifiers do the features in this specification create or expose to the web?

None.

> 14. How does this specification distinguish between behavior in first-party and third-party contexts?

Opening an always-on-top popup requires the opener document to be allowed to use the [`window-management` permissions policy](https://w3c.github.io/window-management/#permissions-policy-integration) and to have the `window-management` permission granted. Third-party iframes cannot open always-on-top popups unless explicitly delegated the `window-management` permissions policy by the embedding first-party document.

> 15. How do the features in this specification work in the context of a browser’s Private Browsing or Incognito mode?

The behavior is the same as in regular browsing mode, except that `window-management` permission grants are not persisted across sessions or inherited from regular browsing profiles.

> 16. Does this specification have both "Security Considerations" and "Privacy Considerations" sections?

Yes. See [Security & Privacy Considerations in the Explainer](EXPLAINER_always_on_top.md#5-security--privacy-considerations).

> 17. Do features in your specification enable origins to downgrade default security protections?

No.

> 18. What happens when a document that uses your feature is kept alive in BFCache (instead of getting destroyed) after navigation, and potentially gets reused on future navigations back to the document?

An always-on-top popup is an independent top-level browsing context decoupled from the opener tab's lifecycle. If the opener document navigates and enters BFCache, the opened always-on-top popup remains open and unaffected. If the document inside the always-on-top popup itself navigates, BFCache behavior follows standard top-level popup navigation rules.

> 19. What happens when a document that uses your feature gets disconnected?

Because `window.open(..., 'alwaysOnTop=true')` requires an active browsing context to open a popup, disconnected documents cannot open always-on-top windows. Once opened, the always-on-top popup is a standalone top-level window and is not closed if the opener document is later disconnected or closed.

> 20. Does your spec define when and how new kinds of errors should be raised?

No new error types are raised. Consistent with existing `window.open()` `windowFeatures` processing, if the `window-management` permission is not granted, if the window is not opened as a popup, or if user agent limits are exceeded, the `alwaysOnTop` feature request is silently ignored and the window opens as a standard popup (or tab) with `window.alwaysOnTopPopup` set to `false`.

> 21. Does your feature allow sites to learn about the user's use of assistive technology?

No.

> 22. What should this questionnaire have asked?

N/A.
