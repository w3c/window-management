## Window Placement Spec & Permission Rename
## Background
The Multi-Screen Window Placement [spec](https://w3c.github.io/window-placement/) has primarily utilized the terminology "Window Placement" ever since it was initially drafted in 2020. This includes the title of the spec, the spec short name (`window-placement`), and user-facing API components like [permission](https://w3c.github.io/window-placement/#api-permission-api-integration) and permission [policy](https://w3c.github.io/window-placement/#api-permission-policy-integration) descriptors which gate the API.

## Problem
The existing "Window Placement" terminology only describes one specific component of the API that allows web applications to [place browser windows on target displays](https://w3c.github.io/window-placement/#api-window-attribute-and-method-definition-changes). It doesn't encompass the entire surface of the API described in the spec, nor will it adequately describe potential future additions to the API.

## Goals
Use more generalized wording that encapsulates more of the current API functionality and provides better futureproofing of the API. Specifically, rename all instances of "Window Placement" terminology with "Window Management". This includes:
*   Repository names and [URLs](https://github.com/w3c/window-placement)
*   [Spec](https://w3c.github.io/window-placement) titles, short names and URLs
*   Spec definitions: [permission](https://w3c.github.io/window-placement/#api-permission-api-integration) and permission [policy](https://w3c.github.io/window-placement/#api-permission-policy-integration) descriptors
    *   Specifically "`window-placement`" will become "`window-management`"
*   Other spec text and descriptions
*   [Web Platform Tests](https://github.com/web-platform-tests/wpt/tree/master/window-placement) and resulting [URLs](https://wpt.live/window-placement/)

## Breaking Changes
A majority of the rename should not be disruptive and URL redirections will be used wherever possible. However renaming the permission and permission policy descriptors require careful migration efforts since they are user-facing strings which are in use by web applications. The following sections provide illustrative examples of what web developers will be required to update. The timeframe of these changes are browser specific; chromium's tentative schedule is outlined in [User Agent API Migration](#user-agent-api-migration)

### Permission Descriptor
Web applications that query the permission name to obtain permission to use the API will need to update:

**From:**
```js
navigator.permissions.query({name:'window-placement'})
```

**To:**
```js
navigator.permissions.query({name:'window-management'})
```
### Permission Policy
Web applications utilizing the [iframe allow attribute](https://w3c.github.io/webappsec-permissions-policy/#iframe-allow-attribute) to control this feature will need to update:

**From:**
```html
<iframe allow="window-placement"> ... </iframe>
```

**To:**
```html
<iframe allow="window-management"> ... </iframe>
```
Web servers utilizing the "[Permissions-Policy](https://w3c.github.io/webappsec-permissions-policy/#permissions-policy-http-header-field)" header to control this feature will need to update:

**From:**
```
Permission-Policy: window-placement 'self'
```

**To:**
```
Permission-Policy: window-management 'self'
```

## Migration Plan
More details on the migration efforts shall be posted and tracked in [Issue #114](https://github.com/w3c/window-placement/issues/114). The following sections provide a high level overview of the migration plan.


### User Agent API Migration
User agents are expected to do their own due diligence in carefully migrating user-facing descriptors. However, a general approach is described below and is the plan Chromium will follow:
1. Add the new descriptor strings "`window-management`" which act as an alias to the old descriptors "`window-placement`", gated behind a browser flag to enable or disable the new aliases.
    * Additionally, the browser should log a console deprecation warning when the old aliases are used by a web application.
    * Chromium ETA: Q4 2022
2. Slowly roll out the new alias across users and track metrics on usage of the old vs. new alias.
    * Chromium ETA: Q4 2022
3. Add a browser flag which removes the old alias.
    * Chromium ETA: Q1 2023
4. Slowly roll out the removal of the old alias.
    * Chromium ETA: Q2 2023

### Spec and Repository
Migration of the repository and spec literature will occur slowly as needed. At a high level, the following will occur or may have already occurred:
*   Rename the permission and permission policy strings in the spec.
*   Update the permission registry with the new permission name.
*   Rename the title of the spec
*   Rename the spec repository, and create redirections from the old URL.
*   Rename the published spec short name and create redirections from the old URL(s).