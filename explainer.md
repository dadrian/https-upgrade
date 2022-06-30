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

* TODO: reference spec change
* TODO: describe opt-out header

## Security and Privacy Considerations

Automatic upgrade is not intended to prevent downgrades. If a site falls back to
HTTP, users will be no worse off than in the status quo.

An active attacker could prevent a link from getting upgraded. This is no worse
off than the status quo, where the link would be accessed over HTTP with no
attempt to load it over HTTPS.

## Risks

TODO

[preload]: https://hstspreload.org