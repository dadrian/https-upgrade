# HTTPS Upgrade - Explainer

Authors: dadrian@google.com, jdeblasio@chromium.org, estark@chromium.org \[your name here]

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

The browser may choose to upgrade active and passive mixed content in addition
to upgrading links. The browser may also choose to upgrade and/or warn the user
before performing POST requests as in form submissions.

The upgrade behavior will require changes to the Fetch spec once the
browser behavior is finalized:

* Step 5 of [Main fetch](https://fetch.spec.whatwg.org/#main-fetch) currently upgrades
requests based on the
[upgrade-insecure-requests](https://w3c.github.io/webappsec-upgrade-insecure-requests/#upgrade-request)
mechanism. This will be replaced to upgrade navigation requests (and possibly subresources)
regardless of the upgrade-insecure-requests mechanism.
* The new step 5 of Main fetch should set a flag on the request to indicate that
an upgrade has been attempted, for http:// requests that were upgraded to https://.
* The [HTTP fetch](https://fetch.spec.whatwg.org/#http-fetch) algorithm should check for
a network error or HTTP error status on upgraded requests. Step 8 of this algorithm
should also be modified to synthesize a redirect from the https:// URL to the http:// URL
and call into HTTP-redirect fetch.

Main fetch should not upgrade requests that have already been upgraded. Edge cases that
will need consideration include: https:// URLs that redirect to http://, whether
subresources need to be differentiated from navigation requests, and how to differentiate
navigation requests that should be upgraded from those that shouldn’t (for example, URLs
that are typed with an explicit “http://” scheme in the browser address bar).


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
