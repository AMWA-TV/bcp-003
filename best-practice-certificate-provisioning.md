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

### Server

### Client


[//]: ## (References)

[//]: ### (Normative)

[RFC-7030]: https://tools.ietf.org/html/rfc7030 "Enrollment over Secure Transport"
