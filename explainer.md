# HTTPS Upgrade - Explainer

Authors: dadrian@google.com, jdeblasio@chromium.org, estark@chromium.org, cthomp@chromium.org \[your name here]

## The problem

Browsers may still make insecure HTTP requests to HTTPS-enabled sites, unless
site operators have performed a series of elaborate configuration steps
culminating in adding their site to the [HSTS preload list][preload]. This can
happen when the user follows an HTTP link to, or loads an HTTP resource from:

* a site that uses HSTS that the browser has not visited before,
* a site that redirects HTTP to HTTPS (defaults to HTTPS) but does not use HSTS,
* or a site that supports both HTTPS and HTTP and does not redirect HTTP to HTTPS.

In all of these cases, users make insecure HTTP connections to sites that support
HTTPS, needlessly compromising their privacy and security. Depending on
configuration, a browser could initiate anywhere between one and all of its requests
to that site over insecure HTTP.

Some browsers ship with lists of sites that are known to support HTTPS, beyond
those already in the HSTS preload list. Maintaining such a list is opaque,
as it requires web crawler data, and error prone, as it will necessarily be out
of date by the time it is shipped to users. It can also be bandwidth intensive,
containing thousands or millions of sites that need to be updated.

## Proposed Behavior Change

The browser should automatically and optimistically upgrade all in-page HTTP links
to HTTPS, with fast fallback to HTTP. 

The browser may optionally respect an opt-out header, which will cause the
browser to consider an HTTPS upgrade as having failed, regardless of status code
or content. This allows web servers that serve different content on HTTP and
HTTPS to prevent autoupgrades.

HTTPS Upgrades are independent of subresource mixed content. The browser should 
continue to upgrade Upgradeable mixed content, and block Blockable mixed content,
based on the security state of the main frame, regardless of if the main frame was
upgraded or not. The browser may also choose to upgrade and/or warn the user
before performing POST requests as in form submissions.

The upgrade behavior will require changes to the Fetch spec once the
browser behavior is finalized:

* A new step will be added after step 5 of
  [Main fetch](https://fetch.spec.whatwg.org/#main-fetch) to upgrade main-frame
  navigation requests from http:// to https://. These upgraded requests will get
  tagged and the original http:// request URL saved (the "fallback URL").
* The [HTTP fetch](https://fetch.spec.whatwg.org/#http-fetch) algorithm should
  check for network error status on upgraded requests. If the request was upgraded
  and the response was a network error, then this algorithm should initiate
  fallback to the saved fallback URL.

Main fetch should not upgrade requests that have already been upgraded. Edge cases that
need consideration include:
* **Subresources**: HTTPS Upgrade only affects main-frame navigations.
  Subresource upgrades are controlled by the user agent's
  policies on mixed content (i.e. autoupgrading passive mixed content). A page
  that is upgraded to HTTPS should follow existing policies for mixed content.
* **URL bar navigations**: Navigations to URLs typed into the URL bar are left
  to the discretion of the user agent, which may already upgrade unschemed
  navigations. User agents may choose to treat URLs entered with an explicit
  "http://" scheme as an "escape hatch" from HTTPS Upgrades.
* **Javascript Navigations**: Navigations via `window.location` are elligible to
  be upgraded.
* **POST requests**: HTTPS Upgrade only affects _idempotent_ requests (i.e.
  `GET`). Forms on upgraded pages should follow existing mixed content policies.
* **Redirects to HTTP**: If a navigation would result in a redirect to HTTP,
  that redirection should also get upgraded to HTTPS. This applies both to
  navigations that are initially to HTTP URLs which get upgraded to HTTPS (and
  then redirected to an HTTP URL) and to navigations that are initially to HTTPS
  URLs that are redirected to HTTP. If these upgrades result in a redirect loop
  (for example, https://http.badssl.com/ redirects to http://http.badssl.com,
  which would be upgraded to https://http.badssl.com/, and so on), this should
  be considered a failed upgrade.

## Security and Privacy Considerations

Automatic upgrade is not intended to prevent downgrades. If a site falls back to
HTTP, users will be no worse off than in the status quo.

An active attacker could prevent a link from getting upgraded. This is no worse
off than the status quo, where the link would be accessed over HTTP with no
attempt to load it over HTTPS. This change does limit information observable to
a purely passive attacker.

Silently upgrading HTTP URLs could have negative effects on the ecosystem by
removing the incentive for developers to fix HTTP references, or making it less
likely that they are aware of them at all. However, with many browsers already
aggressively marking HTTP pages as "Not secure", it is unclear what additional
incentives could be used to get developers to fix HTTP links that could be HTTPS.
Further, many such websites might be unmaintained or infrequently updated. By
attempting to upgrade navigations to HTTPS, browsers protect users' privacy on
websites that wouldn't ever get updated to point to HTTPS directly.

## Risks

If a large number of sites serve different content on HTTPS than HTTP,
automatically upgrading links may make it difficult for a user to access the
"correct" version of a page. [Previous work][levin-upgrades] from 2020 estimates
1-3% of sites have at least 1 page that differs between HTTP and HTTPS. It is
unclear how that translates into percentage of total page loads, or how that
translates into visible user impact.

Content equivalence is difficult to measure. Browsers will have to rely on
feedback from experimentation to determine the impact of unwanted upgrades. The
opt-out header may be used by site administrators to prevent unwanted upgrades.

[preload]: https://hstspreload.org
[levin-upgrades]: https://www.cs.umd.edu/~dml/papers/https_tma20.pdf
