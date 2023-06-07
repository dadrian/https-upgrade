# [Self-Review Questionnaire: Security and Privacy](https://w3ctag.github.io/security-questionnaire/)

> 01.  What information does this feature expose,
>      and for what purposes?

This feature does not expose new information.

> 02.  Do features in your specification expose the minimum amount of information
>      necessary to implement the intended functionality?

Yes (this feature does not expose information).

> 03.  Do the features in your specification expose personal information,
>      personally-identifiable information (PII), or information derived from
>      either?

No.

> 04.  How do the features in your specification deal with sensitive information?

N/A

> 05.  Do the features in your specification introduce state
>      that persists across browsing sessions?

Yes, optionally UAs may remember that an upgrade for a host has previously failed, and skip future upgrades for some time. (Currently, Chrome remembers these in an "HTTP Allowlist" for 7 days for non-Incognito sessions.) Remembering these decisions _could_ be used as a fingerprinting vector, but would require (1) bouncing the user through many main frame navigation redirects, and (2) decorating the URL to track each result, so this risk reduces to the general bounce-tracking/link-decoration issue which is not unique to this feature and we do not consider to be a meaningful new risk.

> 06.  Do the features in your specification expose information about the
>      underlying platform to origins?

No.

> 07.  Does this specification allow an origin to send data to the underlying
>      platform?

No.

> 08.  Do features in this specification enable access to device sensors?

No.

> 09.  Do features in this specification enable new script execution/loading
>      mechanisms?

No.

> 10.  Do features in this specification allow an origin to access other devices?

No.

> 11.  Do features in this specification allow an origin some measure of control over
>      a user agent's native UI?

No.

> 12.  What temporary identifiers do the features in this specification create or
>      expose to the web?

None.

> 13.  How does this specification distinguish between behavior in first-party and
>      third-party contexts?

N/A -- HTTPS-Upgrades only apply to main frame navigations.

> 14.  How do the features in this specification work in the context of a browser’s
>      Private Browsing or Incognito mode?

This feature can also apply in Private Browsing or Incognito mode, at the discretion of the UA.

> 15.  Does this specification have both "Security Considerations" and "Privacy
>      Considerations" sections?

Yes -- we have these sections in our [explainer](https://github.com/dadrian/https-upgrade/blob/main/explainer.md#security-and-privacy-considerations).

> 16.  Do features in your specification enable origins to downgrade default
>      security protections?

No.

> 17.  What happens when a document that uses your feature is kept alive in BFCache
>      (instead of getting destroyed) after navigation, and potentially gets reused
>      on future navigations back to the document?

N/A as this is not a feature exposed to pages.

Pages in BFCache can be reused based on their final committed state without issue with this feature (e.g., a navigation that successfully upgraded to HTTPS previously should be committed to history as HTTPS and the HTTPS page can be directly restored from the BFCache).

> 18.  What happens when a document that uses your feature gets disconnected?

N/A
