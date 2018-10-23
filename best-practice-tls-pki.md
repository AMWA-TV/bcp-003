# [Work In Progress] AMWA Best Current Practice for use of TLS and PKI with NMOS APIs.

## Scope

This document specifies how to secure communications used for HTTP and WebSocket communications within NMOS APIs.

This is based on best practice used for HTTPS, and is intended to promote a secure approach to interoperability.

The contents of this document are suitable for inclusion in a "full stack" specification,
and specify what is required of the network environment, required behaviour of (Media) Nodes, and of clients.

The recommendations are also suitable for other APIs beyond NMOS.

Use of insecure communication (plain HTTP etc.) is forbidden within the scope of this document.

Securing video and audio transport is not in scope.

Use of HTTP/2 is not in scope (although is discussed in [BBC-WHP337])

## Normative References

[RFC 8446](https://datatracker.ietf.org/doc/rfc8446/): 
Transport Layer Security 1.3

[RFC 5280](https://datatracker.ietf.org/doc/rfc5280): 
Internet X.509 Public Key Infrastructure Certificate
and Certificate Revocation List (CRL) Profile

[RFC 6797](https://datatracker.ietf.org/doc/rfc6797/):
HTTP Strict Transport Security (HSTS) 

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
- a connection controll applciation using the IS-05 Connection API

_Note: In the parts of (Full Stack) that concern IS-04 and IS-05 "Media Node" has a similar meaning._

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
-  The Server (and optionally Client) presents X.509 certificates signed by a point of mutual trust.
  - This allows for identification of the parties.

When used correctly HTTPS provides an excellent level of security.
However it is important it is implemented well, with up-to-date versions,
and in a way that will ensure cross vendor inter-operability.

> Note: the text will initially say TLS 1.2 but will be updated!

A later document will cover **authorisation**, i.e. how the Server can determine whether the Client should be allowed to carry out the requested operation.



## Certificate Authority

There SHALL be at least one X.509 Certificate Authority (CA) available.

This may be local or remote.

_Something about looking after keys?_


## X.509 Certificates

Certificates shall conform with X.509 v3 [RFC 5280]


## TLS

### TLS Versions

Implementations SHOULD support TLS 1.3. 
If this is not possible they SHALL support TLS 1.2. 

Note: TLS 1.3 has only recently been finalised, so is not yet mandatory here.
However, implementors should be ready to upgrade,
and TLS 1.2 may be deprecated in a future version of this best practice.

Implementations SHALL not use TLS 1.0 and 1.1. These are deprecated.

Implemenations SHALL NOT use SSL. 
Although the SSL protocol has previously been used to secure HTTP traffic, 
no version of SSL is now considered secure.


### TLS Cipher Suites

_Copy from security-proposal.md, update for TLS 1.3._

## Server Behaviour

### Certificates

Servers SHALL provide a means of installing X.509 certificates.

Servers SHOULD allow multiple certificates, to support Elliptic Curve Ciphers. 

Servers SHALL provide a means of installing a Root CA.

- Some more text about CA chains?

### HTTP

See [TLS](#TLS) for 

Servers SHALL accept HTTP over TLS requests.

Servers SHALL NOT accept plain HTTP requests.

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

CORS?

### WebSockets

Servers that implement WebSocket connections SHALL use TLS.

### Other protocols

Where possible, TLS should be used 




### DNS-SD

Servers SHOULD use unicast DNS-SD, not multicast DNS-SD to advertise their API endpoints.  

Servers SHALL NOT advertise 

_The Full Stack draft makes this a SHALL._

However, ...


## Client Behaviour

### Certificates

### HTTP

### WebSockets

### Other Protocols

### DNS-SD

Clients SHOULD use unicast DNS-SD in preference to multicast DNS-SD to find API endpoints from a server.

## Other considerations

### DHCP

The Full Stack draft requires the Network Environment to provide DHCP.
As it is an inherently insecure protocol, it is insufficient to secure
an environment without the other provisions of this document. 

### DNS

The Full Stack draft requires the Network Environment to provide DNS.
As it is an inherently insecure protocol, it is insufficient to secure
an environment without the other provisions of this document. 


## Recommendations for future NMOS APIs

Read the OWASP Cheat Sheets.

## Further reading

The IETF RFCs referenced here provide much more information.

The [OWASP-REST] Security Cheat Sheet provides further guidance for API implementers.
- With even more information in [OWASP-TRANSPORT], [OWASP-XML], and other Cheat Sheets.

BBC R&D White Papers 337 and 338 provide more information about securing NMOS APIs with TLS and PKI, and many references to online resources and test tools.

_Does this match what is actually in the NMOS documentation?_

## References (informative)


[BBC-WHP337](https://www.bbc.co.uk/rd/publications/whitepaper337)

[BBC-WHP338](https://www.bbc.co.uk/rd/publications/whitepaper338)

[OWASP-REST](https://www.owasp.org/index.php/REST_Security_Cheat_Sheet)