# Security & Privacy of Initiating Multi-Screen Experiences

The following considerations are taken from the [W3C Security and Privacy
Self-Review Questionnaire](https://www.w3.org/TR/security-privacy-questionnaire).

The answers pertain to the [Initiating Multi-Screen Experiences](https://github.com/w3c/window-placement/blob/main/EXPLAINER_initiating_multi_screen_experiences.md) enhancement. That explainer contains pertinent 
[Security Considerations](https://github.com/w3c/window-placement/blob/main/EXPLAINER_initiating_multi_screen_experiences.md#security-considerations) and 
[Privacy Considerations](https://github.com/w3c/window-placement/blob/main/EXPLAINER_initiating_multi_screen_experiences.md#privacy-considerations) sections.

See [security_and_privacy.md](https://github.com/w3c/window-placement/blob/main/security_and_privacy.md) for answers pertaining to the underlying [Multi-Screen Window Placement API](https://www.w3.org/TR/window-placement).

## 2.1 What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?
This feature enhancement does not expose any additional information; see the base API's [corresponding answer](https://github.com/w3c/window-placement/blob/main/security_and_privacy.md#21-what-information-might-this-feature-expose-to-web-sites-or-other-parties-and-for-what-purposes-is-that-exposure-necessary).
## 2.2 Do features in your specification expose the minimum amount of information necessary to enable their intended uses?
This feature enhancement does not expose any additional information; see the base API's [corresponding answer](https://github.com/w3c/window-placement/blob/main/security_and_privacy.md#22-is-this-specification-exposing-the-minimum-amount-of-information-necessary-to-power-the-feature).
## 2.3 How do the features in your specification deal with personal information, personally-identifiable information (PII), or information derived from them?
No additional PII is collected; see the base API's [corresponding answer](https://github.com/w3c/window-placement/blob/main/security_and_privacy.md#23-how-does-this-specification-deal-with-personal-information-or-personally-identifiable-information-or-information-derived-thereof).
## 2.4 How do the features in your specification deal with sensitive information?
This feature enhancement does not expose any additional sensitive information; see the base API's [corresponding answer](https://github.com/w3c/window-placement/blob/main/security_and_privacy.md#24-how-does-this-specification-deal-with-sensitive-information).
## 2.5 Do the features in your specification introduce new state for an origin that persists across browsing sessions?
This feature enhancement does not introduce any additional state that persists across browser sessions. The proposed capability to open a new companion window is limited by the [transient activation duration](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation-duration), and fullscreen companion windows left open when a browsing session ends may be restored by the user agent at the start of the next browsing session, in the same manner as any other windows managed by the user agent. See the base API's [corresponding answer](https://github.com/w3c/window-placement/blob/main/security_and_privacy.md#25-does-this-specification-introduce-new-state-for-an-origin-that-persists-across-browsing-sessions)
## 2.6 Do the features in your specification expose information about the underlying platform to origins?
No additional information is exposed; see the base API's [corresponding answer](https://github.com/w3c/window-placement/blob/main/security_and_privacy.md#26-what-information-from-the-underlying-platform-eg-configuration-data-is-exposed-by-this-specification-to-an-origin).
## 2.7 Does this specification allow an origin to send data to the underlying platform?
No; the use of the platform's window manager APIs to open and place windows using this enhancement is essentially equivalent to the use of the same platform APIs without this enhancement.
## 2.8 Do features in this specification enable access to device sensors?
No additional access is enabled; see the base API's [corresponding answer](https://github.com/w3c/window-placement/blob/main/security_and_privacy.md#27-does-this-specification-allow-an-origin-access-to-sensors-on-a-users-device).
## 2.9 Do features in this specification enable new script execution/loading mechanisms?
No; see the base API's [corresponding answer](https://github.com/w3c/window-placement/blob/main/security_and_privacy.md#29-does-this-specification-enable-new-script-executionloading-mechanisms).
## 2.10 Do features in this specification allow an origin to access other devices?
No additional access is allowed; see the base API's [corresponding answer](https://github.com/w3c/window-placement/blob/main/security_and_privacy.md#210-does-this-specification-allow-an-origin-to-access-other-devices).
## 2.11 Do features in this specification allow an origin some measure of control over a user agent’s native UI?
No; see the base API's [corresponding answer](https://github.com/w3c/window-placement/blob/main/security_and_privacy.md#211-does-this-specification-allow-an-origin-some-measure-of-control-over-a-user-agents-native-ui).
## 2.12 What temporary identifiers do the features in this specification create or expose to the web?
None; see the base API's [corresponding answer](https://github.com/w3c/window-placement/blob/main/security_and_privacy.md#212-what-temporary-identifiers-might-this-specification-create-or-expose-to-the-web).
## 2.13 How does this specification distinguish between behavior in first-party and third-party contexts?
The capability is currently limited to a same-frame context, whether that pertains to the first-party top-level context, or a third-party context of an embedded frame. The frame making the initial fullscreen request will only be granted the proposed capability when it has the required permission (and permission policy). See the base API's [corresponding answer](https://github.com/w3c/window-placement/blob/main/security_and_privacy.md#213-how-does-this-specification-distinguish-between-behavior-in-first-party-and-third-party-contexts).
## 2.14 How do the features in this specification work in the context of a browser’s Private Browsing or Incognito mode?
The behavior should be the same as for regular mode, except that the user agent should not persist permission data and should request permission every session. See the base API's [corresponding answer](https://github.com/w3c/window-placement/blob/main/security_and_privacy.md#214-how-does-this-specification-work-in-the-context-of-a-user-agents-private--browsing-or-incognito-mode).
## 2.15 Does this specification have both "Security Considerations" and "Privacy Considerations" sections?
Yes; see the base API's [corresponding answer](https://github.com/w3c/window-placement/blob/main/security_and_privacy.md#215-does-this-specification-have-a-security-considerations-and-privacy-considerations-section).
## 2.16 Do features in your specification enable origins to downgrade default security protections?
No; see the base API's [corresponding answer](https://github.com/w3c/window-placement/blob/main/security_and_privacy.md#216-does-this-specification-allow-downgrading-default-security-characteristics).
## 2.17 How does your feature handle non-"fully active" documents?
This capability can only be triggered from a "fully active" document.
## 2.18 What should this questionnaire have asked?
Perhaps "What usable-security considerations does this feature raise? (e.g. risks related to spoofing or phishing)". This topic is considered thoroughly in the proposal's [Security Considerations](https://github.com/w3c/window-placement/blob/main/EXPLAINER_initiating_multi_screen_experiences.md#security-considerations) section.
