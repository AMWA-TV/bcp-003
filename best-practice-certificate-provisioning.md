# [Work In Progress] Best Practice Certificate Provisioning

<!-- TOC -->
[[Work In Progress] Best Practice Certificate Provisioning](#work-in-progress-best-practice-certificate-provisioning)
- [Scope](#scope)
- [Use of Normative Language](#use-of-normative-language)
- [Normative References](#normative-references)
- [Definitions](#definitions)
  - [API](#api)
<!-- /TOC -->

## Scope

This document specifies how to implement automated provisioning of TLS Server Certificates to NMOS APIs.

This is based on best practice used for RESTful APIs, and is intended to promote a secure approach to interoperability.

Use of insecure communication (plain HTTP etc.) is forbidden within the scope of this document.

Implementation of [BCP-003-01](best-practice-secure-comms.md) is a recommended prerequisite to implementing this document.

Although security of web pages presented to users is also important, this is outside the scope of this document, which is concerned only with APIs.

## Use of Normative Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][RFC-2119].

## Normative References

These appear at the end of the Markdown source for this document,
and are referenced as hyperlinks within the main body.

## Definitions

_See also the [NMOS Glossary](https://github.com/AMWA-TV/nmos/wiki/Glossary), and definitions within RFCs._

### API

An HTTP / WebSocket API as defined in an AMWA NMOS Specification (IS-04, IS-05, IS-06, etc.)

### CSR (Certificate Signing Request)

The syntax used to request a TLS Certificate from a Certificate Authority, it contains the public key and identification information require by the Certificate Authority in the PKCS Format [RFC-2986][RFC-2986]

### EST Server

The entity that is providing the EST API in accordance with [RFC 7030][RFC-7030]

### EST Client

The entity that is using the EST API, for example:

- a Node requesting a TLS Server Certificate
- a Node requesting the latest Root CA of the domain

### NMOS Server

The entity that is providing an NMOS API, for example:

- a registry implementing IS-04 Registration and Query APIs
- a Node implementing IS-04 Node API and IS-05 Connection API

### NMOS Client

The entity that is using the NMOS API, for example:

- a Node using the IS-04 Registration API
- a monitoring application using the IS-04 Query API
- a connection control application using the IS-05 Connection API

## Introduction (informative)

This document covers the automated provisioning of TLS Server Certificates to NMOS Servers, which are then used to secure communications between NMOS Servers and NMOS Clients.

This document is not concerned with the security of the connection used to carry out provisioning of the TLS Server Certificate, but for the mechanism described in this document to be effective the connection must be secured, ideally using the recommendation covered in [BCP-003-01](best-practice-secure-comms.md).

## Automated Certificate Provisioning Flow (informative)

1. Before the NMOS Node(EST Client) is shipped from the factory it must be provisioned with a unique TLS Client Certificate, signed by the Manufacturers Certificate Authority
2. When the EST Client is connected to the target environments network, it will first discover the location of the EST Server using Unicast DNS-SD.
    * The EST Client should assume the EST server found using DNS-SD is trusted
3. The EST Client should then request the Root CA for the target network, from the EST Server
    * Using the Root CA returned to secure further communications with the EST Server and NMOS Servers
4. The EST Client must generate a Certificate Signing Request(CSR) for all supported TLS Cipher Suites with appropriate key lengths
    * Included in the CSR must be the DNS resolvable name for the current domain of the EST Client
    * Appropriate values for the remaining CSR fields SHOULD be used
5. The EST Client should send each CSR to the EST Server
    * The EST Client must provide the Manufacturer issued certificate during the TLS handshake, if valid
    * The EST Server will use the TLS Client Certificate to validate whether the EST Client is authorized to be issued with a TLS Server Certificate. If the EST CLient is validated the EST Server will return a signed TLS Server Certificate
6. The EST Client should provide the TLS Server Certificate and the corresponding chain of trust for all future HTTPS request to its NMOS APIs

The workflow if no valid Manufacturer TLS Client Certificate is present or if EST Client is moved to a new network are covered in detail in the following [sections](#est-client).

## DNS-SD Advertisement

The EST Server MUST be advertised using unicast DNS-SD as per [RFC 6763][RFC-6763].
The EST Server SHOULD NOT be advertised via mDNS-based DNS-SD.
EST Clients MUST not trust a mDNS-based DNS-SD advertisement for the EST server.
The Authorization Server MUST be advertised with the following service type:

```
_nmos-certs._tcp
```

The hostname and port of the EST Server MUST be identified via the DNS-SD advertisement, with the full HTTPS path then being resolved via the use of the path-prefix of `/.well-known/` as defined in [RFC5785][RFC-5785] and the registered name of `est`. Thus, a valid EST server URI path begins with `https://www.example.com/.well-known/est`. A DNS A record MUST be provided to allow the hostname to be resolved.

Multiple DNS-SD advertisements for the same API are permitted where the API is exposed via multiple ports and/or protocols.

EST Clients MUST support discovering the EST Server through use of unicast DNS-SD service discovery, as described in [RFC 6763][RFC-6763].

### DNS-SD TXT Records

#### api_ver

The DNS-SD advertisement MUST be accompanied by a TXT record of name 'api_ver'. The value of this TXT record is a comma-separated list of API versions supported by the server. For example: 'v1.0,v1.1,v2.0'. There should be no whitespace between commas, and versions should be listed in ascending order.

#### pri

The DNS-SD advertisement MUST include a TXT record with key 'pri' and an integer value. Servers MAY additionally present a matching priority via the DNS-SD SRV record 'priority' and 'weight' as defined in [RFC 2782][RFC-2782]. The TXT record should be used in favour of the SRV priority and weight where these values differ, in order to overcome issues in the Bonjour and Avahi implementations. Values 0 to 99 correspond to an active EST Server API (zero being the highest priority). Values 100+ are reserved for development work to avoid colliding with a live system.

## EST Server Behaviour

### EST Server API

The EST Server SHALL present an instance of the EST API [RFC 7030][RFC-7030].

This MUST include the following API endpoints:

| Operation                       | Operation path  |
| ------------------------------- | --------------- |
| Distribution of CA Certificates | /cacerts        |
| Enrolment of Clients            | /simpleenroll   |
| Re-enrolment of Clients         | /simplereenroll |


### EST Server Authentication

The EST Server MUST present a valid TLS Server Certificate signed by the CA for realm from which it is issuing certificates.

### EST Server Client Authentication

The EST Server MUST authenticate the EST Client that is requesting a TLS Certificate manually or automatically using a TLS Client Certificate.

The EST Server MUST support using a TLS Client Certificate, presented during the TLS handshake by the EST Client to authenticate if the Client is trusted. The TLS Client Certificate can either be signed my the current CA or a trusted Root CA of the device manufacturer.

The EST Server MAY also support manually authentication of the EST Client if no TLS Client Certificate is presented during the TLS handshake, the TLS Client Certificate is not trusted or as an extra authentication step. The exact process for manual authentication will be implementation specific, but the EST Server MUST provide enough information to the user so they can authenticate the EST Client.

## EST Client

### Initial Certificate Provisioning

### Certificate Renewal

### Expired TLS Server Certificate

### Connection to new network

## Further Reading

The IETF RFCs referenced here provide much more information.

[//]: ## (References)

[//]: ### (Normative)

[RFC-2119]: https://tools.ietf.org/html/rfc2119
"Key words for use in RFCs to Indicate Requirement Levels"

[RFC-2617]: https://tools.ietf.org/html/rfc2617
"HTTP Authentication: Basic and Digest Access Authentication"

[RFC-2782]: https://tools.ietf.org/html/rfc2782
"A DNS RR for specifying the location of services (DNS SRV)"

[RFC-2986]: https://tools.ietf.org/html/rfc2986
"PKCS #10: Certification Request Syntax Specification"

[RFC-5785]: https://tools.ietf.org/html/rfc5785
"Defining Well-Known Uniform Resource Identifiers (URIs)"

[RFC-6763]: https://tools.ietf.org/html/rfc6763
"DNS-Based Service Discovery"

[RFC-7030]: https://tools.ietf.org/html/rfc7030
"Enrollment over Secure Transport"

[//]: ### (Informative)
