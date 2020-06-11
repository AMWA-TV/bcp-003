# AMWA BCP-003-02: Authorization in NMOS Systems \[Work In Progress\]

Please see the IS-10 Specification at [NMOS Authorization][IS-10] for further details regarding implementing
authorization with the NMOS suite of APIs.

## Use of Normative Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in [RFC 2119][RFC-2119].

## Scope

The IS-10 Specification has outlined the specification for implementing a general OAuth 2.0 authentication system, in
line with current best practices. This document outlines additional requirements placed upon Resource Servers which
implement specific NMOS Interface Specifications.

## NMOS APIs

### IS-04 - Discovery and Registration

When registering resources with an instance of an [IS-04 Registry][], the Registry MUST capture the Client ID of the
client performing the initial 'node' resource type registration. The Client ID can be found within the `client_id` claim
of the valid JWT access token used in the HTTP request. Where the `client_id` is not specified the same identifier can
be found in the `azp` claim.

Subsequent requests to register, modify or delete resources which belong to Node ID(s) which were registered by this
client MUST be validated to ensure that the Client ID used matches the `client_id` or `azp` claim in the JWT access
token used in the HTTP request making the modifications. This is to ensure that clients do not, maliciously or
incorrectly, alter resources belonging to other Nodes. Unsuccessful validation of a matching Client ID MUST result in
the Resource server rejecting the request with a '401' HTTP response code. All IS-04 resources can be related back to a
single Node ID by following the [IS-04 Referential Integrity][] guide.

[IS-10]: https://amwa-tv.github.io/nmos-authorization/branches/v1.0-dev/ "AMWA IS-10 NMOS Authorization API"
[RFC-2119]: https://tools.ietf.org/html/rfc2119 "Key words for use in RFCs to Indicate Requirement Levels"

[IS-04 Registry]: http://amwa-tv.github.io/nmos-discovery-registration/ "AMWA IS-04 NMOS Discovery and Registration Specification"

[IS-04 Referential Integrity]: https://amwa-tv.github.io/nmos-discovery-registration/tags/v1.3/docs/4.1._Behaviour_-_Registration.html#referential-integrity "AMWA IS-04 Resource Referential Integrity"
