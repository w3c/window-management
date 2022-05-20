# Multi-Screen Window Placement on the Web - Initiating Multi-Screen Experiences

## Introduction

This proposal introduces a Multi-Screen Window Placement API enhancement that
would allow web applications to initiate a multi-screen experience from a single
user activation.

## Background

[Multi-Screen Window Placement](https://github.com/webscreens/window-placement)
is a new permission-gated web platform API that provides web applications with
information about the device's connected screens, allows them to open and
position windows or request fullscreen on any of those screens.

The HTML living standard specifies
[Tracking User Activation](https://html.spec.whatwg.org/multipage/interaction.html#tracking-user-activation),
to gate some web platform APIs on user interaction signals. The transient user
activation signal (e.g. from a mouse click) is required and consumed when web
application scripts open a popup window via Window.open(), and when they request
fullscreen via Element.requestFullscreen().

## Problem

Web applications wishing to offer compelling multi-screen experiences are
currently hindered by the single-screen paradigms incorporated in existing
algorithms of standard specifications.

A Multi-Screen Window Placement API partner requested the ability to display
multi-screen content when the user clicks a button. See
[Issue 1233970: Window Placement: Pop-ups are blocked when fullscreen is requested](http://crbug.com/1233970).

Specifically, they would like to
[place fullscreen content on a specific screen](https://webscreens.github.io/window-placement/#usage-overview-place-fullscreen-content-on-a-specific-screen)
*and*
[place a new popup window on a separate specific screen](https://webscreens.github.io/window-placement/#usage-overview-place-windows-on-a-specific-screen),
but each of these consume the click’s activation signal, and the second script
request is blocked (unless the user grants popups without gestures via a broad
‘Popups and Redirects’ content setting).

## Use Cases

The aim of this proposal is enable better experiences for web application users
with multiple screens. Here are some use cases that inform the goal below:
* Slideshow app presents fullscreen slides and opens a speaker notes window
* Financial app launches a fullscreen dashboard and a stock tracker window
* Medical app shows a fullscreen x-ray image and opens an image list window
* Creativity app enters a fullscreen preview and opens a separate control window

## Goals

The goal of this document is to explore tractable and incremental solutions for
the stated multi-screen web application use cases, while minimizing any facets
of the capability that would be prone to abuse. The following specific goals and
proposed solutions make incremental extensions of existing window placement APIs
to support initiating a multi-screen experience from a single user gesture.
* Extend `Element.requestFullscreen()`, `Window.open()`, and
`Tracking User Activation` algorithms and concepts to conditionally permit a
fullscreen request and a popup window request from a single user-activation.

An eventual end-goal (that is not explored here) is to enable scripts to enter
fullscreen on N screens and open M popup windows on M screens from a single user
activation, where N and M are disjoint and potentially empty sets, as long as
the site has the multi-screen `window-placement` permission.

## Proposal

Allow sites with the `window-placement` permission to
[place fullscreen content on a specific screen](https://webscreens.github.io/window-placement/#usage-overview-place-fullscreen-content-on-a-specific-screen)
*and*
[place a new popup window on a separate specific screen](https://webscreens.github.io/window-placement/#usage-overview-place-windows-on-a-specific-screen),
from a single user gesture, when the device has multiple screens and fullscreen
targets a specific screen.

This fulfills the API partner’s use case in the most narrow manner possible, and
limits the potential for accidental misuse or abuse with the following set of
mitigations:

- Require the window-placement permission and devices with multiple displays
  - These are needed to show a popup and fullscreen on separate displays
- Allow opening a popup *after* requesting fullscreen on a specific display
  - This ordering prevents basic misuse (showing a popup under fullscreen)
  - Pre-existing protections prevent placing popups over fullscreen
    - Chrome exits fullscreen (see
    [ForSecurityDropFullscreen](https://source.chromium.org/chromium/chromium/src/+/main:content/public/browser/web_contents.h;l=1203?q=ForSecurityDropFullscreen))
    - Firefox exits fullscreen
    - Safari opens the popup in a separate fullscreen space
- Do not automatically activate popups opened from a fullscreen frame
  - This leaves input focus on the fullscreen window, for “Press [Esc] to exit”
- Activate popups once the opener exits fullscreen
  - This avoids creation of persistent popunders (e.g. Chrome’s
  [PopunderPreventer](https://source.chromium.org/chromium/chromium/src/+/main:chrome/browser/ui/blocked_content/popunder_preventer.h))

This proposal leverages the flexibility of implementation-specific behavior
(blocking popups, postponing window activation), while enabling a path towards
defining standardized algorithms. It is partly inspired by existing standardized
precedent: requestFullscreen()
[waives its activation requirement](https://fullscreen.spec.whatwg.org/#dom-element-requestfullscreen)
on user generated orientation changes.

Note: the `window-placement` permission is requested earlier, as needed, by
[`Window.getScreenDetails()`](https://w3c.github.io/window-placement/#dom-window-getscreendetails),
before scripts can request fullscreen (or open a popup) on a specific screen.
So it is expected that scripts making these requests already have permission.

### Spec Changes

This proposal can be brought about as algorithmic changes to existing specs:
- [Tracking User Activation](https://html.spec.whatwg.org/multipage/interaction.html#tracking-user-activation)
- [`Element.requestFullscreen()`](https://fullscreen.spec.whatwg.org/#dom-element-requestfullscreen)
- [The rules for choosing a browsing context](https://html.spec.whatwg.org/multipage/browsers.html#the-rules-for-choosing-a-browsing-context-given-a-browsing-context-name)
  - Invoked by [`Window.open()`](https://html.spec.whatwg.org/multipage/window-object.html#dom-open-dev)'s
  [window open steps](https://html.spec.whatwg.org/multipage/window-object.html#window-open-steps) 

Loosely, an internal slot can be added to the window context, activated when a
fullscreen request is made that targets a specific screen of a multi-screen
device, checked and consumed in the rules for choosing a browsing context, when
the document requests openining a new popup window.

### Open questions

#### Feature detection

There is no obvious way for a web application to detect support for the proposed
feature, in advance of attempting its use. Scripts can attempt to use this
functionality and check the return value of the call to `Window.open()`, to
determine whether the window was created successfully or not.

It may be possible to expose signals for this transient token akin to the
[proposed JS API for querying User Activation](https://github.com/dtapuska/useractivation).

See the existing algorithm for
[requesting new browsing contexts](https://html.spec.whatwg.org/multipage/browsers.html#the-rules-for-choosing-a-browsing-context-given-a-browsing-context-name),
invoked from the
[window.open steps](https://html.spec.whatwg.org/multipage/window-object.html#window-open-steps),
which may permit or block popup window requests based on the
[transient activation](https://html.spec.whatwg.org/multipage/interaction.html#transient-activation)
signal, and the user agent’s configuration.

## Alternatives Considered

- Permit N `Window.open()` calls on a device with N screens
  - This doesn't satisfy the goal of also entering fullscreen
- Introduce a new API to support multi-screen fullscreen
  - This is a desirable long-term path, but it is not nearly as tractable
- Allow a target-screen fullscreen request after opening a cross-screen popup
  - While similar to the active proposal, this has fewer inherent protections
  against accidental misuse or abuse regarding popunders
