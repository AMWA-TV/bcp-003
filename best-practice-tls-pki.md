# [Work In Progress] AMWA Best Current Practice for use of TLS and PKI with NMOS APIs.

## Scope

This document specifies how to secure communications used for HTTP and WebSocket communications within NMOS APIs.

This is based on best practice used for HTTPS, and is intended to promote a secure approach to interoperability.

The contents of this document are suitable for inclusion in a "full stack" specification,
and specify what is required of the network environment, required behaviour of (Media) Nodes, and of clients.

The recommendations are also suitable for other APIs beyond NMOS.

## Normative References

[RFC 8446](https://datatracker.ietf.org/doc/rfc8446/): 
Transport Layer Security 1.3

[RFC 5280](https://datatracker.ietf.org/doc/rfc5280): 
Internet X.509 Public Key Infrastructure Certificate
and Certificate Revocation List (CRL) Profile

## Definitions

_See also the NMOS Glossary, and definitions within RFCs._

### API

An HTTP / WebSocket API as defined in an AMWA NMOS Specification (IS-04, IS-05, IS-06, etc.)

### API Server

The entity that is providing the API, for example:

- a registry implementing IS-04 Registration and Query APIs
- a Node implementing IS-04 Node API and IS-05 Connection API.

### API Client

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

The API Server or API Client that is sending API Messages.

_Note: this corresponds to "Sender" in RFC 8446, but that term has a specific meaning in AMWA IS-04 and IS-05._

### Message Receiver

The API Server or API Client that is receiving API Messages.

_Note: this corresponds to "Receiver" in RFC 8446, but that term has a specific meaning in AMWA IS-04 and IS-05._

## Introduction (informative)

_See [Definitions](#Definitions) for explanation of terms used here._

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


## Network Environment

_Note: this corresponds to a section of the Full Stack draft._

### DHCP

The Full Stack draft requires the Network Environment to provide DHCP.
As it is an inherently insecure protocol, it is insufficient to secure
an environment without the other provisions of this document. 

### DNS

The Full Stack draft requires the Network Environment to provide DNS.
As it is an inherently insecure protocol, it is insufficient to secure
an environment without the other provisions of this document. 


### Certificate Authority

There SHALL be at least one X.509 Certificate Authority (CA) available to both Servers and Clients.

This may be local or remote.




...

## Server Behaviour

### HTTP

Servers SHALL accept HTTP over TLS requests.

Servers SHALL NOT accept plain HTTP requests.

### WebSockets

Servers that implement WebSocket connections SHALL use TLS.

### Other protocols

Where possible, TLS should be used 


### DNS-SD

Servers SHOULD use unicast DNS-SD to advertise their API endpoints,
as multicast DNS-SD is more .  

_The Full Stack draft makes this a SHALL._

However, 

### TLS Versions

Implementations of Servers and Clients SHOULD support TLS 1.3. 
If this is not possible they SHALL support TLS 1.2. 

Note: TLS 1.3 has only recently been finalised, so is not yet mandatory here.
However, implementors should be ready to upgrade,
and TLS 1.2 may be deprecated in a future version of this best practice.

Implemenations SHALL not use TLS 1.0 and 1.1. These are deprecated.

Implementations SHALL NOT use SSL. 
Although the SSL protocol has previously been used to secure HTTP traffic, 
no version of SSL is now considered secure.


### TLS Cipher Suites

### Certificates

Servers and Clients must provide at least one well-documented way of allowing users to install certificates.

Certificates shall be X.509 v3 [RFC 5280]


This must be used for all web services (HTTPS interactions, WSS connections etc). This must be X.509 v3 certs and must allow more than one cert to be installed [this is important when using Elliptic Curve ciphers]


### Use of CA Certificates


## Clients

### Use of Root CA Certificates

### Authenticating Communications


## DNS-SD

Clients SHOULD use unicast DNS-SD in preference to multicast DNS-SD to find API endpoints from a server.

## Other recommendations

BBC R&D White Papers 337 and 338 provide more information about securing NMOS APIs with TLS and PKI.

The following recommendations are based on [OWASP-REST]:

- Reject all requests not explcitly allowed by the API with HTTP response code 405 Method not allowed
  - The allowed requests are typically defined using RAML

_Does this match what is actually in the NMOS documentation?_

## References (informative)


[BBC-WHP337](https://www.bbc.co.uk/rd/publications/whitepaper337)

[BBC-WHP338](https://www.bbc.co.uk/rd/publications/whitepaper338)

[OWASP-REST](https://www.owasp.org/index.php/REST_Security_Cheat_Sheet)