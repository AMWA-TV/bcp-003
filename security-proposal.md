# AMWA NMOS APIs Security Proposal

The AMWA NMOS APIs IS-04, IS-05 and IS-06 are an important building
block of the future of IP broadcast production. Between them they
provide an open, cross vendor inter-operable solution for discovery and
registration, connection management, and network control. These APIs
build on existing web technologies such as HTTP, REST and JSON. One of
the advantages of this approach is that we can now harness tried and
proven web security technologies to secure these APIs.

In order for this work to succeed it is important to consider the
concepts of confidentiality, identification, integrity, authentication
and authorisation:

Confidentiality

- Data passing between client and the APIs is unreadable to
    third parties.

Identification

- The client can check whether the API it is interacting with is owned
    by a trusted party.

Integrity

- It must be clear if data travelling to or from the API been
    tampered with.

Authentication

- The client can check if packets actually came from the API it is
    interacting with, and vice versa.

Authorisation

- The API can determine whether the client interacting with it has
    authorisation to carry out the operation requested.

This document will outline a method of achieving these goals using a
combination of HTTPS, OAuth2 and JSON Web Tokens.

## Connection Security - HTTP over TLS (HTTPS)

A secure implementation of the AMWA NMOS APIs is dependent on the
existence of a secure connection between the API server and its client.
On the web HTTP over TLS (HTTPS) is widely used for ensuring security of
HTTP communications, providing the tools to deliver confidentiality,
identification, integrity and authentication in one protocol. As the
NMOS APIs use HTTP they can already be used over HTTPS, but further work
is required to ensure this is done in a secure and interoperable way.

HTTPS uses the TLS protocol to tunnel HTTP over an encrypted connection.
This protects the connection from interception, providing
confidentiality of data. Signed hashes of the content are included in
the message, providing both message integrity and authentication. As
only the sender’s secret key can sign the message hash this
authenticates the packet as having originated with the sender, with the
hash itself allows the receiver to check that content has not been
altered. Finally X509 certificates are presented by the server (and
optionally the client) which are signed by a point of mutual trust. This
allows for identification of the parties participating in the
connection.

When used correctly HTTPS provides an excellent level of security.
However it is important it is implemented well and in a way that will
ensure cross vendor inter-operability. More information on the following
recommendations can be found in [BBC R&D White Paper
337](https://www.bbc.co.uk/rd/publications/whitepaper337).

### TLS Versions

The TLS protocol has four versions. TLS 1.0 and 1.1 are deprecated, and
should not be used by NMOS API implementations. TLS 1.2 is currently in
wide usage and is secure providing it is correctly configured, and must
be supported when using TLS with the NMOS APIs. TLS 1.3 is currently a
proposed draft which is expected to be published at some point during
2018. Once published use of TLS 1.3 will be best security practice, and
implementers of NMOS APIs and client should be ready to support it.

Note that in the past the SSL protocol has been used to secure HTTP
traffic. No version of SSL is considered secure, and it must not be used
to secure NMOS API implementations.

### TLS 1.2 Cipher Suites

When using TLS 1.2 with NMOS APIs should use the cipher suites listed in
Appendix 1 in the priority order in which they are
listed. Where possible servers should be configured to honour this
ordering. Clients should support all the cipher suites listed, but must
as a minimum support enough to ensure reliable interoperability.

Devices heavily constrained by resources may omit RSA certificate based
TLS. This does not compromise security, but does reduce backward
compatibility. It is important to ensure that the device will be
operating in an environment where ECDHE certificates are available,
which is not the case for all corporate certificate authorities.
Omitting RSA certificates results in the list given in
the Appendix 2.

Where resources are extremely limited, NMOS APIs may choose to support
the cipher suite in Appendix 3 only. As a result NMOS
API clients must support this cipher to ensure interoperability.

Further work will be required to select cipher suites for use with TLS
1.3 once it has been formally published.

### Server Configuration for TLS 1.2

Servers hosting secure NMOS APIs should:

- Disable SSL Compression

- Use HSTS (RFC 6797) where appropriate (i.e secure HTTP is always
    in use)

- Not use Public Key Pinning

- Consider Disabling TLS Session Tickets

## Public Key Infrastructure Considerations

API server and client implementers should consider how their
implementations will interact will Public Key Infrastructure. In
particular implementers must provide a mechanism for deploying and
revoking subject certificates for API servers. API clients must provide
a way or installing root certificates to API clients to establish a root
of trust.

## Client Identification and Authorisation

HTTPS is the ideal technology for securing the connection between the
client and API server, and also for identifying the server to the
client. HTTPS is well proven in these roles. However, while it does have
some capability to perform client identification and authorisation,
HTTPS does not provide the fine grained control over permissions desired
for the NMOS APIs.

*JSON Web Tokens* are access tokens which can be issued to a client to
enable them to carry out certain actions on APIs. A tokens is issued to
the client by an authentication server upon successful authentication of
the client. The client can then use that token in requests it sends to
APIs. The token contains *claims* which describe what actions the client
is allowed to do against the API, for example reading or writing to a
particular endpoint. The set of claims a particular token contains is
referred to as being its *scope*. A particular token can contain many
different claims, allowing it to be used for a variety of actions.

Tokens are cryptographically signed, to allow their provinance to be
validated. Various algorithms can be used for this, but RSA with SHA-256
should be used as the signing algorithm for NMOS APIs to ensure
compatibility.

### Client Authentication

Before tokens can be issued to an API client it must first authenticate
with the authentication server. Clients would use *OAuth 2.0* to
authenticate. OAuth 2.0 is a widely used authorisation mechanism which
has been used to provide authentication for issuing JSON Web Tokens on
other systems.

![Token acquisition - image courtesy of Riedel<span
data-label="fig:tokenAquisition"></span>](images/nmos_sec_1.png)

The token request and grant process is illustrated above.
This diagram shows a client system
requesting an access token which it could use to, for example, control
IS-05 interfaces on the Node. As the diagram shows, passwords must be
sent to the authentication server. As such it is absolutely vital that
HTTPS is in use as it is important that these passwords cannot be
intercepted.

Once authentication has been successfully completed, and the
authentication server is happy the client has permissions required for
the scope of token it has requested, it will issue the token to the
client. Tokens are time limited, and must be renewed with the
authentication server once they expire. This allows permissions to be
revoked by system administrators if required.

The digram also shows two NMOS Nodes retrieving the
public key from a /certs endpoint. They may use this to check the
signature on any JSON web tokens they receive, to ensure that they are
genuine. This operation need only be completed once during the lifetime
of the key used by the authentication server to sign the tokens.

The authentication server advertises itself using DNS-SD in the same
manner as the IS-04 APIs. As such client devices may automatically
discover the authentication server they should use on the network
without manual configuration.

### Client Authorisation

Once a client has obtained a web token from the the authentication
server it may then use that token in API calls it makes. The token is
placed in the header of the request, but otherwise the request is the
same as it would normally be when using the API.

As JSON Web Tokens are not encrypted themselves, it is important that
these requests are also performed over HTTPS. If not encrypted, the Web
Token could be intercepted and used to make calls by a third party.

![Client authorisation - image courtesy of Riedel<span
data-label="fig:clientAuthorisation"></span>](images/nmos_sec_2.png)

The process of using the Web Token is illustrated above.
On receiving a Web Token for the
first time the API server will check the validity of the token with the
authentication server’s public key. The API server will then check that
the scope of the web token is sufficient to carry out the operation
requested, and if it is it will then carry out the API request and
respond as normal. API servers do not need to contact the authentication
server directly to authenticate the token, provided they have previously
obtained its public key.

## Future Work

In order for this to be implemented the following work is required:

- Definition of claims required for operations on existing AMWA APIs.

- Specification of the mDNS/DNS record advertisements used to
    advertise the authentication server.

Summary

This document has laid out a mechanism for securing the AMWA NMOS APIs
in the following manner:

- HTTP over TLS 1.2 to secure all API connections.

- Be ready for changes introduced in TLS 1.3

- Make use of cipher suites listed in the appendix.

- Use of an authentication server to issue JSON Web Tokens.

- Authentication of clients with the authentication server using
    OAuth 2.0.

- API permissions controlled by the scope of JSON Web Tokens.

## Apendix

### TLS 1.2 Cipher Suite Lists

#### Full Cipher Suite Set

This set of cipher suites should be supported by NMOS API servers unless
hardware limitations make this impractical.

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

TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CCM\_8

#### ECDSA only Cipher Suite Set

This set of cipher suites may be supported by NMOS API servers not
requiring wishing to reduce hardware resource by supporting only ECDSA
certificates.

TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_GCM\_SHA256

TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_GCM\_SHA384

TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CBC\_SHA256

TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_CBC\_SHA384

TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CCM\_8

#### CCM Based Cipher Suite

This cipher suite must be supported by all NMOS clients and servers
regardless of hardware capability.

TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CCM\_8
