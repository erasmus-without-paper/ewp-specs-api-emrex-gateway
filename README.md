EMREX Gateway API
=================

* [What is the status of this document?][statuses]
* [See the index of all other EWP Specifications][develhub]


Summary
-------

This document describes the **EMREX Gateway API**. It is an "extension" to
[EMREX NCP API][emrex-ncp-api]. Existing EMREX NCP endpoints may implement this
API to allow their clients to use additional authentication and encryption
methods used by EWP APIs.


Introduction
------------

Originally, EMREX has been designed to handle solely anonymous clients. The
student himself decided whether to trust the client, and whether to share
his/her data with that client. However, it turned out that some NCP endpoints
need more fine-grained access control (e.g. to allow commercial use).

One of the solutions [discussed][future-of-emrex-webinar] was to design the
EMREX Gateway API described here. This step would also be a first step towards
[joining EMREX and EWP registers][joining-registers].

Note, that you are NOT required to implement EWP's EMREX Gateway API in order
to make use of EMREX:

 * You MAY be an EMREX client without joining EWP Network. However, some EMREX
   providers (NCPs) may require their clients to authenticate themselves. At
   the moment of writing this, joining EWP Network is the only way for your
   client to do so.

 * You MAY implement an EMREX provider (NCP) without joining EWP Network.
   However, in this case, all of your client requests will remain anonynous.
   If you want to restrict access to your NCP API based on who the client is,
   you will need to do both - join EWP Network *and* implement the *EMREX
   Gateway API*.

You will need to be familiar with both EWP and EMREX before you read on.


About "NCP direct" manifest entry
---------------------------------

At the time of writing this, EMREX maintainers haven't defined EWP manifest
entry for their EMREX NCP API. To allow smooth transition from existing EMREX
implementations (the ones that don't make use of EWP), we need to do that for
them.

That's why this specification defines **two** manifest entries, not one
(you can see them in the [`manifest-entries.xsd`'][manifest-entries.xsd] file).
This additional API entry allows the EWP Registry Service to include servers
which *do* implement the original EMREX NCP API, but *do not* implement EMREX
Gateway API (and this, in turn, allows newer EMREX clients to discover them
without making a separate request to EMREG).

In practice, you will probably not be required to explicitly specify that you
implement the "NCP direct" API in your manifest file. Instead, it is planned
for EWP Registry Service to dynamically do that for you, when it detects that
your institution is listed as an NCP implementer in EMREG. This allows all
EMREX implementers to be listed in EWP Registry *even before they join EWP*.
You should consult EWP Registry API specification to confirm that these plans
came through. (Both specifications - this one, and the Registry API - should be
updated if it does.)


Request method
--------------

 * Requests MUST be made with either HTTP GET or HTTP POST method. Servers MUST
   support both these methods. Servers SHOULD reject all other request methods.

 * Clients are advised to use POST when passing large number of parameters
   (servers MAY set a limit on their GET query string length).


Request parameters
------------------

Parameters MUST be provided in the regular `application/x-www-form-urlencoded`
format.


### `hei_id` (required)

An institution identifier. ID of the HEI which the client wants to retrieve
ToRs from.

Servers MUST be able to accept all HEI IDs declared in their [manifest
files][discovery-api] (the ones covered by this API).

Note some differences between EMREX Gateway API and the original EMREX NCP API:

* Original NCPs could cover either:

  - all HEI in a single country, or
  - exactly one HEI.

  In the first case, the client wasn't able to specify in which institutions he
  was interested in - he would always receive combined information from all
  HEIs in this country (covered by this NCP).

* In EWP's model, APIs can cover any number of HEIs. This may be a single HEI,
  all HEIs in a single country, or any other set of HEIs (also in multiple
  countries).

  EMREX Gateway API attempts to find a common ground between EMREX and EWP
  models by no longer attempting to group HEIs by country. In other words, all
  NCP APIs wrapped with EMREX Gateway API become "singleFetch" (as defined in
  EMREX specification).

* Original NCP API did not take the `hei_id` parameter, nor any similar
  parameter. However, EMREX Gateway API requires the client to provide it, and
  requires the server to respect it. This may require some internal changes in
  existing country-wide NCPs (i.e. results filtering based on this HEI).


### All other NCP parameters (required)

This specification "wraps" all the request parameters described in the
[original EMREX NCP API specification][emrex-ncp-api]. The request MUST contain
all the required parameters as defined in the original API.

Note, that in the original EMREX, it was the user's browser that used to make
the request. It's no longer necessarily true in case of EMREX Gateway API
(because the user's browser is probably not able to handle EWP Security
methods, nor the server is required to handle AJAX requests).

Also note, that EMREX APIs are not being versioned. There's only one version of
EMREX NCP API - the current one. By implementing *EMREX Gateway API* in version
*1.x.x* you state that your requests are compatible with the current version of
EMREX NCP API. (Hopefully, EMREX NCP API will remain backward compatible in
order to allow that.)


Security
--------

This version of this API uses [standard EWP Authentication and Security,
Version 2][sec-v2]. Server implementers choose which security methods they
support by declaring them in their Manifest API entry.


Handling of invalid parameters
------------------------------

 * General [error handling rules][error-handling] apply.

 * Invalid (unknown) `hei_id` values MUST be **ignored**. Servers MUST return
   a valid (HTTP 200) XML response in such cases, but the response will simply
   not contain the information on the unknown `hei_id` values. If all values
   are unknown, servers MUST respond with an empty `<response>` element.
   This requirement is true even when `<max-hei-ids>` is equal to `1`.

 * If the length of `hei_id` list is greater than `<max-hei-ids>`, servers
   MUST respond with HTTP 400.


Response
--------

Servers MUST respond with a valid XML document described by the
[response.xsd](response.xsd) schema. See the schema annotations for further
information.


[emrex-ncp-api]: https://confluence.csc.fi/display/EMREX/Implementation+details%3A+NCP
[future-of-emrex-webinar]: https://confluence.csc.fi/display/EMREX/Webinar+on+the+future+of+a+combined+register)
[develhub]: http://developers.erasmuswithoutpaper.eu/
[statuses]: https://github.com/erasmus-without-paper/ewp-specs-management#statuses
[discovery-api]: https://github.com/erasmus-without-paper/ewp-specs-api-discovery
[joining-registers]: https://github.com/erasmus-without-paper/general-issues/issues/29
