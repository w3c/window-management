# [Self-Review Questionnaire: Security and Privacy](https://w3ctag.github.io/security-questionnaire/)

> 01.  What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?

The feature exposes the display state of the window (minimized/maximized/normal/fullscreen), an event fired when the display state changes, and whether the window is user resizable.

All of these are necessary for better native integration of VDI web clients.

> 02.  Do features in your specification expose the minimum amount of information necessary to enable their intended uses?

Yes.

> 03.  How do the features in your specification deal with personal information, personally-identifiable information (PII), or information derived from them?

N/A.

> 04.  How do the features in your specification deal with sensitive information?

N/A.

> 05.  Do the features in your specification introduce new state for an origin that persists across browsing sessions?

No.

> 06.  Do the features in your specification expose information about the underlying platform to origins?

No.

> 07.  Does this specification allow an origin to send data to the underlying platform?

No.

> 08.  Do features in this specification enable access to device sensors?

No.

> 09.  Do features in this specification enable new script execution/loading mechanisms?

No.

> 10.  Do features in this specification allow an origin to access other devices?

No.

> 11.  Do features in this specification allow an origin some measure of control over a user agent's native UI?

No.

> 12.  What temporary identifiers do the features in this specification create or expose to the web?

Display state of the window (minimized/maximized/fullscreen/normal).

> 13.  How does this specification distinguish between behavior in first-party and third-party contexts?

It doesn't distinguish.

> 14.  How do the features in this specification work in the context of a browser’s Private Browsing or Incognito mode?

Same as in regular mode.

> 15.  Does this specification have both "Security Considerations" and "Privacy Considerations" sections?

Yes.

> 16.  Do features in your specification enable origins to downgrade default security protections?

No.

> 17.  How does your feature handle non-"fully active" documents?

Events aren't fired at them; they are saved for later and fired in a single coalesced event.

> 18.  What should this questionnaire have asked?

N/A.
