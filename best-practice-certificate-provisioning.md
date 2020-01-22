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

Although security of web pages presented to users is also important,
this is outside the scope of this document, which is concerned only with APIs.

## Use of Normative Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][RFC-2119].

## Normative References

These appear at the end of the Markdown source for this document,
and are referenced as hyperlinks within the main body.

## Definitions

_See also the [NMOS Glossary](https://github.com/AMWA-TV/nmos/wiki/Glossary), and definitions within RFCs._

### API

An HTTP / WebSocket API as defined in an AMWA NMOS Specification (IS-04, IS-05, IS-06, etc.)

### EST Server

### NMOS Server

The entity that is providing an NMOS API, for example:

- a registry implementing IS-04 Registration and Query APIs
- a Node implementing IS-04 Node API and IS-05 Connection API

### EST Client

The entity that is using the EST API, for example:

- a Node requesting a TLS Server Certificate

### CSR (Certificate Signing Request)

## Introduction (informative)

## Automated Certificate Provisioning Specification

### EST Server API

The EST Server SHALL present an instance of the EST API [RFC 7030][RFC-7030].

This MUST include the following API endpoints:

| Operation                       | Operation path  |
| ------------------------------- | --------------- |
| Distribution of CA Certificates | /cacerts        |
| Enrollment of Clients           | /simpleenroll   |
| Re-enrollment of Clients        | /simplereenroll |



### DNS-SD Advertisement

The EST Server Must advertised using unicast DNS-SD as per [RFC 6763][RFC-6763].
The EST Server SHOULD NOT be advertised via mDNS-based DNS-SD.
The Authorization Server MUST be advertised with the following service type:

```
_nmos-certs._tcp
```

The hostname and port of the EST Server MUST be identified via the DNS-SD advertisement, with the full HTTPS path then being resolved via the use of the path-prefix of `/.well-known/` as defined in [RFC5785][RFC5785] and the registered name of `est`. Thus, a valid EST server URI path begins with `https://www.example.com/.well-known/est`. A DNS A record MUST be provided to allow the hostname to be resolved.

Multiple DNS-SD advertisements for the same API are permitted where the API is exposed via multiple ports and/or protocols.

EST Clients MUST support discovering the EST Server through use of unicast DNS-SD service discovery, as described in [RFC 6763][RFC-6763].

#### DNS-SD TXT Records

##### api_ver

The DNS-SD advertisement MUST be accompanied by a TXT record of name 'api_ver'. The value of this TXT record is a comma-separated list of API versions supported by the server. For example: 'v1.0,v1.1,v2.0'. There should be no whitespace between commas, and versions should be listed in ascending order.

##### pri

The DNS-SD advertisement MUST include a TXT record with key 'pri' and an integer value. Servers MAY additionally present a matching priority via the DNS-SD SRV record 'priority' and 'weight' as defined in [RFC 2782][RFC-2782]. The TXT record should be used in favour of the SRV priority and weight where these values differ, in order to overcome issues in the Bonjour and Avahi implementations. Values 0 to 99 correspond to an active EST Server API (zero being the highest priority). Values 100+ are reserved for development work to avoid colliding with a live system.



[//]: ## (References)

[//]: ### (Normative)

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
