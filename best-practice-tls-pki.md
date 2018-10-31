# [Work In Progress] AMWA Best Current Practice for use of TLS and PKI with NMOS APIs

[//]: # (ToC goes after this comment. Generate with gen-toc.sh <this-file> and paste in.)

- [Scope](#scope)
- [Normative References](#normative-references)
- [Definitions](#definitions)
  - [API](#api)
  - [Server](#server)
  - [Client](#client)
  - [Message](#message)
- [Introduction (informative)](#introduction-informative)
- [TLS](#tls)
  - [TLS Versions](#tls-versions)
  - [TLS 1\.3 Cipher Suites](#tls-13-cipher-suites)
  - [TLS 1\.2 Cipher Suites](#tls-12-cipher-suites)
  - [X\.509 Certificates and Certificate Authority](#x509-certificates-and-certificate-authority)
- [Server Behaviour](#server-behaviour)
  - [Certificate Management: Server](#certificate-management-server)
  - [HTTP: Server](#http-server)
  - [WebSockets: Server](#websockets-server)
  - [Other Protocols: Server](#other-protocols-server)
  - [DNS\-SD: Server](#dns-sd-server)
- [Client Behaviour](#client-behaviour)
  - [Certificate Management: Client](#certificate-management-client)
  - [HTTP: Client](#http-client)
  - [WebSockets: Client](#websockets-client)
  - [Other Protocols: Client](#other-protocols-client)
  - [DNS\-SD: Client](#dns-sd-client)
- [Other Considerations](#other-considerations)
  - [DHCP](#dhcp)
  - [DNS](#dns)
- [Recommendations for Future Interface Specifications](#recommendations-for-future-interface-specifications)
- [Further Reading](#further-reading)

[//]: # (ToC goes before this comment)

## Scope

This document specifies how to secure communications used for HTTP and WebSocket communications within NMOS APIs.

This is based on best practice used for HTTPS, and is intended to promote a secure approach to interoperability.

The recommendations are also suitable for other APIs beyond NMOS.

Use of insecure communication (plain HTTP etc.) is forbidden within the scope of this document.

Fine-grained Client authorisation --
providing a mechanism to determine what actions a Client may take against an API --
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

### Message

Information sent according to an NMOS API, e.g.:

- an HTTP request
- an HTTP response
- a WebSocket message

A Message can also carry payload data related to an NMOS API.

- For example the draft IS-07 specification uses WebSocket and MQTT transport to carry event data.
  These can be considered as Messages for the purposes of this document.
  - Although ST 2110 payload information is not considered a Message here.

## Introduction (informative)

The AMWA NMOS Interface Specfications use HTTP and WebSockets for API communications between Nodes,
network services and control applications.

This document identifies best practice for providing these communications with:

- **Confidentiality**:
    Messages passing between the Client and Server is unreadable to third parties.

- **Identification**:
    The Client can check whether the Server is owned by a trusted party.

- **Integrity**:
    It must be clear if Messages have been tampered with.

- **Authentication**:
    The Client can check if Messages actually came from the Server it is
    interacting with, and vice versa

This is achieved as follows:

- HTTP and WebSocket communications are tunneled over TLS (i.e. they use HTTPS and WSS).
- The Server or Client sending each Message includes in it a signed hash to authenticate that it is the originator.
- The other party checks the hash to check that the Message has not been altered.
- The Server (and optionally Client) presents X.509 certificates, preferably signed by a Certificate Authority
  - This provides a point of mutual trust to identify the parties

A later document will cover **authorisation**,
i.e. how the Server can determine whether the Client should be allowed to carry out the requested operation.

When used correctly HTTPS provides an excellent level of security.
However it is important it is implemented well, with up-to-date versions,
and in a way that will ensure cross vendor inter-operability.
These recommendations only provide an overview of this rapidly-changing field,
and readers should see [Further Reading](#further-reading) for more detail,
and information about test software and other resources.

## TLS

### TLS Versions

Implementations SHOULD support TLS 1.3 and SHALL support TLS 1.2.

Note: TLS 1.3 has only recently been finalised, so is not yet mandatory here.
However, implementors should be ready to upgrade, as 1.3 may be mandatory in a future revision.

Implementations SHALL NOT use TLS 1.0 or 1.1. These are deprecated.

Implementations SHALL NOT use SSL.
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

A Certificate Authority (CA) SHOULD be available to sign certificates.

- This provides a point of trust for the environment.

The CA root certificate SHALL be available to Servers and Clients.

Implementers SHOULD consider using a trusted CA service;
if a self-managed CA is used it is important to keep its private key very safe.

If a CA cannot be provided, then "self-signed" certificates MAY be exchanged
directly between Clients and Servers. This SHOULD NOT be used for large installations.

Wildcard certificates SHOULD NOT be used.

There SHALL be a way of revoking Certificates that are no longer needed or compromised.

The CA SHOULD support OCSP requests as per [RFC 6960][RFC-6960]
and OCSP stapling (see so that Clients can check whether certificates are compromised.)

## Server Behaviour

### Certificate Management: Server

Servers SHALL provide a means of installing X.509 certificates.
These SHOULD be signed by the CA, unless "self-signed" certificates are being used

- See comments above.

Servers SHALL support installation of multiple certificates,
and SHALL support both RSA and ECDSA certificates.

- ECDSA certificates are suited to the hardware-limited cases discussed above.

Servers SHALL provide a secure mechanism to install and store the private key(s) 
and key chain for their certificates.

It SHOULD be possible for a user to perform the above operations.

### HTTP: Server

Servers SHALL accept and respond to HTTPS requests,
using a TLS version and cipher suite allowed by [TLS](#tls)

Servers SHALL NOT accept or respond to plain HTTP requests.

Servers SHOULD use the Strict-Transport-Security header as per [RFC 6797]
to declare that they only will communicate with secure connections.

Servers SHALL reject all requests not explicitly allowed by the API
with HTTP response code 405 Method not allowed.

- NMOS Specifications typically define the allowed requests using RAML

Servers SHALL validate all request payloads and reject those that are invalid
with HTTP response code xxx.

- NMOS Specifications typically define the allowed payloads using JSON Schema
  - This includes, for example, checking string inputs with regexps.

Servers SHOULD check requests are not too large (HTTP response 413)

Servers SHOULD log invalid requests, to help check for broken/malicious clients.

Servers:

- SHOULD NOT use SSL compression
- SHOULD NOT use Public Key Pinning
- SHOULD NOT use TLS Session Tickets

_TODO: add more detail to the above_

Servers SHOULD be as specific as possible in the use of CORS (cross-origin resource sharing).

- The examples in the IS-04 and IS-05 documentation are "very relaxed",
  and SHOULD NOT be used without considering whether they are appropriate.

### WebSockets: Server

This section applies to Servers providing Messages through a WebSocket connection,
for example for subscription to an AMWA IS-04 Query API.

- It SHOULD also be used to secure WebSocket connections for transport of data,
  for example AMWA IS-07 events.

Servers SHALL provide an encrypted WebSocket connection (wss: URL scheme),
using a TLS version and cipher suite allowed by [TLS](#tls)

Note: this is default for IS-04 WebSocket subscriptions when using HTTPS for the Query API.

Servers SHALL NOT provided unencrypted WebSocket connection (ws: URL scheme).

### Other Protocols: Server

Other protocols used for Messages SHOULD be secured using TLS, where this is supported.

- For instance, MQTT supports use of TLS 1.2.
  - This is an alternative transport for IS-07 events.

Security of protocols where TLS is not available is outside the scope of this document.
Security of ST 2110 streams is outside the scope of this document.

### DNS-SD: Server

Servers SHALL support unicast DNS-SD to advertise their API endpoints.  

Servers SHOULD NOT advertise multicast DNS-SD, except where a DNS server is not available.

- In practice, a DNS server can be expected to be available for "engineered networks".
  Multicast DNS-SD can be useful for small or temporary networks, but presents a security risk.
  Servers SHALL NOT advertise using multicast DNS-SD outside the local network.
  - As of version v1.2, AMWA IS-04 includes DNS-SD announcements of Node APIs.
    However, these may be deprecated and removed from later versions of the spec.

## Client Behaviour

### Certificate Management: Client

Clients SHALL provide a means of installing a root certificate,
and SHALL use this to check the validity of Server certificates.

Clients SHALL provide a way of removing root certificates.

Client SHOULD use OCSP Stapling to identify revoked certificates.

- For TLS 1.2 this is defined in [RFC 6961][RFC-6961]
- For TLS 1.3 it is included in the main spec [RFC 8446][RFC-8446]

It SHOULD be possible for a user to perform these operations.

- Having to return equipment to the manufacturer is not acceptable.
  Having to install firmware updates is undesirable.

_What do we want to say on client certs?_

### HTTP: Client

Clients SHALL make API requests using HTTPS,
using a TLS version and cipher suite allowed by [TLS](#tls)

Clients SHALL NOT make API requests using plain HTTP.

Clients SHALL NOT continue communication with a Server after a failed handshake,
except with the express permission of the user.

- This is similar to the "Add Exception" that web browsers present.
  If the user wishes to continue it is at his/her own risk.
  Clients SHOULD allow a system administrator the option to disable such exceptions.

### WebSockets: Client

This section applies to Clients requesting WebSocket connections as part of an API,
for example for subscription to an AMWA IS-04 Query API,
or WebSocket connections for transport of data, for example AMWA IS-07 events.

Clients SHALL require encrypted WebSocket connections (wss:),
using a TLS version and cipher suite allowed by [TLS](#tls).

Clients SHALL NOT use unencrypted WebSocket connections (ws:).

### Other Protocols: Client

Other protocols used for Messages SHOULD be secured using TLS, where this is supported.

- For example MQTT (see comments re: Server).

### DNS-SD: Client

Clients SHOULD use unicast DNS-SD in preference to multicast DNS-SD to find API endpoints from a Server.

- However, this does not make unicast DNS-SD a substitute for a secure API.
  For instance, if the Server fails to provide a valid Certificate, the Client must not use its endpoint.
  
Clients SHOULD NOT rely on DNS-SD announcements of Node API endpoints for correct operation.

- These may be deprecated and removed from later versions of the spec.


## Other Considerations

### DHCP

In most cases DHCP will be available on the network. However, it is an
insecure protocol and should not be considered as means of providing security
without the other provisions of this document.

### DNS

In most cases DNS will be available on the network. However in many cases
it should be considered insecure, and should not be considered as means of providing security
without the other provisions of this document.

- See also comments about DNS-SD [above](#dns-sd-client)

Secure deployment of DNS is currently outside the scope of this document.

## Recommendations for Future Interface Specifications

Creators of new AMWA Interface Specifcations SHOULD ensure that the recommendations
of this document are included in the Specification itself.

Organisers of interoperabily testing of new Specifications SHOULD include tests of
whether implementations meet the recommendations of this document.

All those involved in creating and testing new Specifications SHOULD be aware of the
general recommendations and "Cheat Sheets" of the
[Open Web Application Security Project (OWASP)](https://www.owasp.org).
These go further than the scope of this document, and cover areas such as
access control, security tokens, audit logs and carriage of sensitive information.
See [Further Reading](#further-reading).

## Further Reading

The IETF RFCs referenced here provide much more information.

OWASP provides a number of useful "Cheat Sheets",
including [REST Security][OWASP-REST] and
[Transport Layer Protection][OWASP-TRANSPORT].

BBC R&D White Papers [337][BBC-WHP337] and [338][BBC-WHP338] provide more information about
securing NMOS APIs with TLS and PKI, and many references to online resources and
test tools. [337][BBC-WHP337] also discusses IPv6.

[//]: ## (References)

[//]: ### (Normative)

[RFC-5280]: https://datatracker.ietf.org/doc/rfc5280/
"Internet X.509 Public Key Infrastructure Certificate and
Certificate Revocation List (CRL) Profile"

[RFC-6797]: https://datatracker.ietf.org/doc/rfc6797/
"HTTP Strict Transport Security (HSTS)"

[RFC-6960]: https://datatracker.ietf.org/doc/rfc6960/
"X.509 Internet Public Key Infrastructure Online Certificate Status Protocol - OCSP"

[RFC-6961]: https://datatracker.ietf.org/doc/rfc6961/
"The Transport Layer Security (TLS) Multiple Certificate Status Request Extension"

[RFC-8446]: https://datatracker.ietf.org/doc/rfc8446/
"Transport Layer Security 1.3"

[//]: ### (Informative)

[BBC-WHP337]: https://www.bbc.co.uk/rd/publications/whitepaper337 "HTTPS Configuration for the AMWA NMOS APIs"

[BBC-WHP338]: https://www.bbc.co.uk/rd/publications/whitepaper338 "Public Key Infrastructure for IP Production for Broadcast"

[OWASP-REST]: https://www.owasp.org/index.php/REST_Security_Cheat_Sheet
"OWASP REST Security Cheat Sheet"

[OWASP-TRANSPORT]: https://www.owasp.org/index.php/Transport_Layer_Protection_Cheat_Sheet
"OWASP Transport Layer Protection Cheat Sheet"
