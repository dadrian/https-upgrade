# HTTPS Upgrade - Explainer

Authors: dadrian@google.com, \[your name here]

## The problem

Browsers may still make insecure HTTP requests to HTTPS-enabled sites, unless
site operators have performed a series of elaborate configuration steps
culminating in adding their site to the [HSTS preload list][preload]. This can
happen when the user follows an HTTP link to, or loads an HTTP resource from: a
site uses HSTS, but the browser has not seen the header; a site redirects HTTP
to HTTPS (defaults to HTTPS) but does not use HSTS; or a site that supports both
HTTPS and HTTP. In all of these cases, users make “needless” insecure HTTP
connections to sites that support HTTPS. This could be a single HTTP request,
one HTTP request per resource, or entirely HTTP, depending on if the site uses
HSTS, _defaults_ to HTTPS, or just _supports_ HTTPS.

Some browsers ship with lists of sites that are known to support HTTPS, beyond
the sites included in the HSTS preload list. Maintaining such a list is opaque,
as it requires web crawler data, and error prone, as it will necessarily be out
of date by the time it is shipped to users. It can also be bandwidth intensive,
containing thousands or millions of sites that need to be updated.

## Proposed Behavior Change

The browser will automatically and optimistically upgrade all in-page HTTP links
to HTTPS, with fast fallback to HTTP. 

The browser may optionally respect an opt-out header, which will cause the
browser to consider an HTTPS upgrade as having failed, regardless of status code
or content. This allows web servers that serve different content on HTTP and
HTTPS to prevent autoupgrades.

The upgrade behavior will require a small change to the fetch spec once the
browser behavior is finalized.

## Security and Privacy Considerations

Automatic upgrade is not intended to prevent downgrades. If a site falls back to
HTTP, users will be no worse off than in the status quo.

An active attacker could prevent a link from getting upgraded. This is no worse
off than the status quo, where the link would be accessed over HTTP with no
attempt to load it over HTTPS.

## Risks

If a large number of sites serve different content on HTTPS than HTTP,
automatically upgrading links may make it difficult for a user to access the
"correct" version of a page. [Previous work][levin-upgrades] from 2020 estimates
1-3% of sites have at least 1 page that differs between HTTP and HTTPS. It is
unclear how that translates into percentage of total pageloads, or how that
translates into visible user impact.

Content equivalence is difficult to measure. Browsers will have to rely on
feedback from experimentation to determine the impact of unwanted upgrades. The
opt-out header may be used by site administrators to prevent unwanted upgrades.

[preload]: https://hstspreload.org