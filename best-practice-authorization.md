# \[Work In Progress\] Best Practice Authorization

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

When registering resources with an instance of an [IS-04 Registry][], the Registry MUST register the Client ID
of the client performing the registration. The Client ID can be found within the `client_id` claim of the
valid JWT access token used in the HTTP request.

Subsequent requests to modify or delete a registered resource MUST validate that the Client ID used to
register the resource matches the `client_id` claim in the JWT access token used in the HTTP request making
the modifications. This is to ensure that clients do not, maliciously or incorrectly, alter resources belonging to
other nodes. Unsuccessful validation of a matching Client ID MUST result in the Resource server rejecting the
request.

[IS-10]: https://amwa-tv.github.io/nmos-authorization/branches/v1.0-dev/ "AMWA IS-10 NMOS Authorization API"
[RFC-2119]: https://tools.ietf.org/html/rfc2119 "Key words for use in RFCs to Indicate Requirement Levels"

[IS-04 Registry]: http://amwa-tv.github.io/nmos-discovery-registration/ "AMWA IS-04 NMOS Discovery and Registration Specification"
