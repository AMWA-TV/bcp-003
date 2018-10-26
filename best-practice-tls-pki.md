# [Work In Progress] AMWA Best Current Practice for use of TLS and PKI with NMOS APIs

[//]: # (ToC goes after this comment. Create it with: ``gh-md-toc --hide-header --depth=3 this-file | sed 's/\* \[/- [/'``)

- [[Work In Progress] AMWA Best Current Practice for use of TLS and PKI with NMOS APIs](#work-in-progress-amwa-best-current-practice-for-use-of-tls-and-pki-with-nmos-apis)
  - [Scope](#scope)
  - [Normative References](#normative-references)
  - [Definitions](#definitions)
    - [API](#api)
    - [Server](#server)
    - [Client](#client)
    - [API Message](#api-message)
    - [Message Sender](#message-sender)
    - [Message Receiver](#message-receiver)
  - [Introduction (informative)](#introduction-informative)
  - [Certificate Authority](#certificate-authority)
  - [X\.509 Certificates](#x509-certificates)
  - [TLS](#tls)
    - [TLS Versions](#tls-versions)
    - [TLS 1\.3 Cipher Suites](#tls-13-cipher-suites)
    - [TLS 1\.2 Cipher Suites](#tls-12-cipher-suites)
  - [Server Behaviour](#server-behaviour)
    - [Certificates](#certificates)
    - [HTTP requests](#http-requests)
    - [WebSockets](#websockets)
    - [Other protocols](#other-protocols)
    - [DNS\-SD](#dns-sd)
  - [Client Behaviour](#client-behaviour)
    - [Certificates](#certificates-1)
    - [HTTP](#http)
    - [WebSockets](#websockets-1)
    - [Other Protocols](#other-protocols-1)
    - [DNS\-SD](#dns-sd-1)
  - [Other considerations](#other-considerations)
    - [DHCP](#dhcp)
    - [DNS](#dns)
  - [Recommendations for future Interface Specifications](#recommendations-for-future-interface-specifications)
  - [Further reading](#further-reading)

[//]: # (ToC goes before this comment)

## Scope

This document specifies how to secure communications used for HTTP and WebSocket communications within NMOS APIs.

This is based on best practice used for HTTPS, and is intended to promote a secure approach to interoperability.

The recommendations are also suitable for other APIs beyond NMOS.

Use of insecure communication (plain HTTP etc.) is forbidden within the scope of this document.

Fine-grained Client authorisation --
providing a mechanism to determin what actions a Client may take against an API --
is not in scope, but may be part of a future revision.

Use of HTTP/2 is not in scope, but may be part of a future revision.

Securing video and audio transport is not in scope.

## Normative References

These appear at the end of the Markdown source for this document,
and are referenced as hyperlinks within the main body.

## Definitions

_See also the NMOS Glossary, and definitions within RFCs._

### API

An HTTP / WebSocket API as defined in an AMWA NMOS Specification (IS-04, IS-05, IS-06, etc.)

### Server

The entity that is providing the API, for example:

- a registry implementing IS-04 Registration and Query APIs
- a Node implementing IS-04 Node API and IS-05 Connection API.

### Client

The entity that is using the API, for example:

- a Node using the IS-04 Registration API
- a monitoring application using the IS-04 Query API
- a connection control application using the IS-05 Connection API

### API Message

Information sent according to an API, e.g.:

- an HTTP request
- an HTTP response
- a WebSocket message
- an MQTT message

### Message Sender

The Server or Client that is sending API Messages.

_Note: this corresponds to "Sender" in RFC 8446, but that term has a specific meaning in AMWA IS-04 and IS-05._

### Message Receiver

The Server or Client that is receiving API Messages.

_Note: this corresponds to "Receiver" in RFC 8446, but that term has a specific meaning in AMWA IS-04 and IS-05._

## Introduction (informative)

The AMWA NMOS Interface Specfications use HTTP and WebSockets for API communications between Nodes, network services and control applications.

This document identifies best practice for providing these communications with:

- **Confidentiality**:
    API Messages passing between the Client and Server is unreadable to third parties.

- **Identification**:
    The Client can check whether the Server is owned by a trusted party.

- **Integrity**:
    It must be clear if API Messages have been tampered with.

- **Authentication**:
    The Client can check if API Messages actually came from the Server it is
    interacting with, and vice versa.

This is achieved as follows:

- HTTP and WebSocket communications are tunneled over TLS (i.e. they use HTTPS and WSS).
- The Message Sender includes a signed hash in the API Message to authticate that it has originated it.
- The Message Receiver uses the hash to check that the API Message has not been altered.
- The Server (and optionally Client) presents X.509 certificates signed by a point of mutual trust.
  - This allows for identification of the parties.

When used correctly HTTPS provides an excellent level of security.
However it is important it is implemented well, with up-to-date versions,
and in a way that will ensure cross vendor inter-operability.

A later document will cover **authorisation**, i.e. how the Server can determine whether the Client should be allowed to carry out the requested operation.

## TLS

### TLS Versions

Implementations SHOULD support TLS 1.3 and SHALL support TLS 1.2.

Note: TLS 1.3 has only recently been finalised, so is not yet mandatory here.
However, implementors should be ready to upgrade, as 1.3 may be mandatory in a future revision.

Implementations SHALL NOT use TLS 1.0 or 1.1. These are deprecated.

Implemenations SHALL NOT use SSL.
Although the SSL protocol has previously been used to secure HTTP traffic,
no version of SSL is now considered secure.

### TLS 1.3 Cipher Suites

Note: TLS allows several different cipher suites;
interoperability requires the Server and Client to support at least one common suite,
which needs to be sufficiently secure.

This section applies to implementations using TLS 1.3. It is consistent with [RFC 8446][RFC-8446].

All Servers and Clients SHALL support this cipher suite:

TLS_AES_128_GCM_SHA256

All Servers SHOULD support the following cipher suites:

TLS_AES_256_GCM_SHA384

TLS_CHACHA20_POLY1305_SHA256

_Note: the above needs discussion by the group._

### TLS 1.2 Cipher Suites

This section applies to implementations using TLS 1.2.

All Servers and Clients SHALL support this cipher suite:

TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CCM\_8

All Servers SHOULD support the following cipher suites,
unless hardware limitations make this impractical.
In such a case Servers SHOULD support the first four listed suites,
which are used with ECDSA certificates.

- Where resources are extremely limited, the mandatory suite above ensures interoperability.

Servers SHOULD be configured to use the priority order listed:

TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_GCM\_SHA256

TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_GCM\_SHA384

TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CBC\_SHA256

TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_CBC\_SHA384

TLS\_ECDHE\_RSA\_WITH\_AES\_128\_GCM\_SHA256

TLS\_ECDHE\_RSA\_WITH\_AES\_256\_GCM\_SHA384

TLS\_DHE\_RSA\_WITH\_AES\_128\_GCM\_SHA256

TLS\_DHE\_RSA\_WITH\_AES\_256\_GCM\_SHA384

TLS\_ECDHE\_RSA\_WITH\_AES\_128\_CBC\_SHA256

TLS\_ECDHE\_RSA\_WITH\_AES\_256\_CBC\_SHA384

TLS\_DHE\_RSA\_WITH\_AES\_128\_CBC\_SHA256

TLS\_DHE\_RSA\_WITH\_AES\_256\_CBC\_SHA256

### X.509 Certificates and Certificate Authority

Implementations SHALL use TLS with X.509 v3 certificates, as per [RFC 5280][RFC-5280].

Certificates SHOULD be signed by a Certificate Authority (CA);
this provides a point of trust for a system.
The CA root certificate SHALL be available to Servers and Clients.

Implementers SHOULD consider using a trusted CA service;
if a self-managed CA is used it is important to keep its private key very safe.

If a CA cannot be provided, then "self-signed" certificates MAY be exchanged
directly between Clients and Servers. This SHOULD NOT be used for large instalations.

Wildcard certificates SHOULD NOT be used.

There SHALL be a way of revoking Certificates that are no longer needed or compromised.

## Server Behaviour

### X.509 Certificates

Servers SHALL provide a means of installing X.509 certificates.

Servers SHALL support installation of multiple certificates,
and SHALL support both RSA and ECDSA certificates.

- ECDSA certificates are suited to the hardware-limited cases discussed above.

Servers SHALL provide a means of installing a Root CA.

Servers SHALL provide a mechanism for revoking certificates.

### HTTP Requests

Servers SHALL accept and respond to HTTPS requests,
using a TLS version and cipher suite allowed by [TLS](#tls)

Servers SHALL NOT accept or respond to plain HTTP requests.

Servers SHOULD use the Strict-Transport-Security header as per [RFC 6797]
to declare that they only will communicate with secure connections.

Servers SHALL reject all requests not explcitly allowed by the API
with HTTP response code 405 Method not allowed.

- NMOS Specifications typically define the allowed requests using RAML

Servers SHALL validate all request payloads and reject those that are invalid
with HTTP response code xxx.

- NMOS Specifications typically define the allowed payloads using JSON Schema
  - This includes, for example, checking string inputs with regexps.

Servers SHOULD check requests are not too large (HTTP response 413)

Servers SHOULD log invalid requests, to help check for broken/malicious clients.

And maybe, Servers:

- SHOULD NOT use SSL compression
- SHOULD NOT use Public Key Pinning
- SHOULD NOT use TLS Session Tickets

Servers SHOULD be as specific as possible in the use of CORS.

- The examples in the IS-04 and IS-05 documentation are "very relaxed", and SHOULD NOT be used without considering whether they are appropriate.

### WebSockets

This section applies to Servers providing WebSocket connections as part of an API,
for example for subscription to an AMWA IS-04 Query API,
or providing a WebSocket connection for transport of data,
for example AMWA IS-07 events.

Servers SHALL provide an encrypted WebSocket connection (wss: URL scheme),
using a TLS version and cipher suite allowed by [TLS](#tls)

Note: this is default for IS-04 WebSocket subscriptions when using HTTPS for the Query API.

Servers SHALL NOT provided unencrypted WebSocket connection (ws: URL scheme).

### Other protocols

Other protocols used in APIs SHOULD be secured using TLS, where this is supported.

- For instance, MQTT supports use of TLS 1.2.
  - This is an alternative transport for IS-07 events.

Security of protocols where TLS is not available is outside the scope of this document.
Security of ST 2110 streams is outside the scope of this document.

### DNS-SD

Servers SHOULD use unicast DNS-SD to advertise their API endpoints.  

_The Full Stack draft makes this a SHALL._

Servers SHOULD NOT use multicast DNS-SD, except where it is not possible to provide a DNS server.

## Client Behaviour

### X.509 Certificates

Clients SHALL provide a means of installing a root certificate.

Clients SHALL check Server certificates against the root certificate.

### HTTP

Clients SHALL make API requests using HTTPS,
using a TLS version and cipher suite allowed by [TLS](#tls)

Clients SHALL NOT make API requests using plain HTTP.

...

### WebSockets

This section applies to Clients requesting WebSocket connections as part of an API,
for example for subscription to an AMWA IS-04 Query API,
or WebSocket connections for transport of data, for example AMWA IS-07 events.

Clients SHALL require encrypted WebSocket connections (wss:),
using a TLS version and cipher suite allowed by [TLS](#tls).

Clients SHALL NOT use unencrypted WebSocket connections (ws:).

...

### Other Protocols

### DNS-SD

Clients SHOULD use unicast DNS-SD in preference to multicast DNS-SD to find API endpoints from a Server.

...

## Other considerations

### DHCP

The Full Stack draft requires the Network Environment to provide DHCP.
As it is an inherently insecure protocol, it is insufficient to secure
an environment without the other provisions of this document.

### DNS

The Full Stack draft requires the Network Environment to provide DNS.
As it is an inherently insecure protocol, it is insufficient to secure
an environment without the other provisions of this document.

## Recommendations for future Interface Specifications

Creators of new AMWA Interface Specifcations SHOULD ensure that the recommendations
of this document are included in the Specification itself.

Organisers of interoperabily testing of new Specifications SHOULD include tests of
whether implementations meet the recommendations of this document.

All those involved in creating and testing new Specifications SHOULD be aware of the
general recommendations and "Cheat Sheets" of the
[Open Web Application Security Project (OWASP)](https://www.owasp.org).
These go futher than the scope of this document, and cover areas such as
access control, security tokensl, audit logs and carriage of sensitive information.
See [Further Reading](#further-reading).

## Further reading

The IETF RFCs referenced here provide much more information.

OWASP provides a number of useful "Cheat Sheets",
including [REST Security][OWASP-REST] and
[Transport Layer Protection][OWASP-TRANSPORT].

BBC R&D White Papers [337][BBC-WHP337] and [338][BBC-WHP338] provide more information about
securing NMOS APIs with TLS and PKI, and many references to online resources and
test tools. [337][BBC-WHP337] also discusses IPv6.

_Does this match what is actually in the NMOS documentation?_

[//]: ## (References)

[//]: ### (Normative)

[RFC-8446]: https://datatracker.ietf.org/doc/rfc8446/
"Transport Layer Security 1.3"

[RFC-5280]: https://datatracker.ietf.org/doc/rfc5280/
"Internet X.509 Public Key Infrastructure Certificate and
Certificate Revocation List (CRL) Profile"

[RFC 6797]: https://datatracker.ietf.org/doc/rfc6797/
"HTTP Strict Transport Security (HSTS)"

[//]: ### (Informative)

[BBC-WHP337]: https://www.bbc.co.uk/rd/publications/whitepaper337 "HTTPS Configuration for the AMWA NMOS APIs"

[BBC-WHP338]: https://www.bbc.co.uk/rd/publications/whitepaper338 "Public Key Infrastructure for IP Production for Broadcast"

[OWASP-REST]: https://www.owasp.org/index.php/REST_Security_Cheat_Sheet
"OWASP REST Security Cheat Sheet"

[OWASP-TRANSPORT]: https://www.owasp.org/index.php/Transport_Layer_Protection_Cheat_Sheet
"OWASP Transport Layer Protection Cheat Sheet"
