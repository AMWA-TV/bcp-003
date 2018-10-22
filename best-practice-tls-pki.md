# [Work In Progress] AMWA Best Current Practice for use of TLS and PKI with NMOS APIs.

## Scope

This document specifies how to secure communications used for NMOS APIs using
Transport Layer Security (TLS) and X.509 Public Key Infrastructure (PKI).
This is based on best practice used for HTTPS, and is intended to promote a secure approach to interoperability.

The contents of this document are suitable for inclusion in a "full stack" specification,
and specify what is required of the network environment, required behaviour of (Media) Nodes, and of clients.

The recommendations are also suitable for other APIs beyond NMOS.

## Definitions

### API

An HTTP / WebSocket API as defined in AMWA IS-04, IS-05, IS-06, etc.

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

Information sent between Server and Client according to an API, e.g.:

- an HTTP request
- an HTTP response
- a WebSocket message

### Message Sender

The entity (Server or Client) that is sending API Messages.

_Note: this corresponds to "Sender" in RFC 8446, but that term has a specific meaning in AMWA IS-04 and IS-05._

### Message Receiver

The entity (Server or Client) that is receiving API Messages.

_Note: this corresponds to "Receiver" in RFC 8446, but that term has a specific meaning in AMWA IS-04 and IS-05._

## Introduction

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


## Network Environment

This applies to any network, not just the "Engineered Networks" considered in (Full Stack) 

### Certificate Authority

There SHALL be at least one X.509 Certificate Authority available. ...

This may be local or remote.

### TLS Configuration

...

## Nodes

### HTTP

Nodes SHALL support use of HTTP over TLS.   

#### Informative note


### WebSockets

Nodes SHALL support use of WebSockets over TLS.

### TLS Versions

Nodes SHALL 


### TLS Cipher Suites

### Installing Certificates

### Use of CA aCertificates

## Clients (including Broadcast Controllers)

### Use of Root CA Certificates

### Authenticating Communitations

## References
