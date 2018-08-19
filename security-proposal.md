# AMWA NMOS APIs Security Proposal

The AMWA NMOS APIs IS-04, IS-05 and IS-06 are an important building
block of the future of broadcast production. These APIs
build on existing web technologies such as HTTP, REST and JSON. One of
the advantages of this approach is that we can harness tried and
proven web security technologies to secure these APIs.

This proposal is designed to meet the following objectives with regard to confidentiality, identification, integrity, authentication
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
On the web, HTTP over TLS (HTTPS) is widely used for ensuring security of
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
altered. Finally X.509 certificates are presented by the server (and
optionally the client) which are signed by a point of mutual trust. This
allows for identification of the parties participating in the
connection.

When used correctly HTTPS provides an excellent level of security.
However it is important it is implemented well and in a way that will
ensure cross vendor inter-operability.

### TLS Versions

The TLS protocol has four versions. TLS 1.0 and 1.1 are deprecated, and
should not be used by NMOS API implementations. TLS 1.2 is currently in
wide usage and is secure, providing it is correctly configured.

TLS 1.2 SHALL be supported when using TLS with the NMOS APIs. TLS 1.3 is currently a
proposed draft which is expected to be published at some point during
2018. Once published use of TLS 1.3 will be best security practice, and
implementers of NMOS APIs and client should be ready to support it.

Note that in the past the SSL protocol has been used to secure HTTP
traffic. No version of SSL is considered secure.

SSL SHALL NOT be used to secure NMOS API implementations.

### TLS 1.2 Cipher Suites

TLS allows several different cipher suites; 
interoperability requires the server and client to support at least one common suite, 
which needs to be sufficiently secure.

This section applies to implementations of NMOS APIS using TLS 1.2. 
This section will be updated when TLS 1.3 is published.

All servers and clients SHALL support this cipher suite:

TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CCM\_8

All servers SHOULD support the following cipher suites,
unless hardware limitations make this impractical.
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

This reduced set of cipher suites MAY be supported by NMOS API servers not
requiring wishing to reduce hardware resource by supporting only ECDSA
certificates (omitting RSA certificates).

TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_GCM\_SHA256

TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_GCM\_SHA384

TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CBC\_SHA256

TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_CBC\_SHA384

This SHOULD only be used where the device is operating in an environment
where ECDHE certificates are available, which is not the case for all
corporate certificate authorities. 

Note: where resources are extremely limited, the mandatory suite
TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_CCM\_8 ensures interoperability.

### Server Configuration for TLS 1.2

Servers hosting secure NMOS APIs should:

- Disable SSL Compression

- Use HSTS (RFC 6797) where appropriate (i.e secure HTTP is always
    in use)

- Not use Public Key Pinning

- Consider Disabling TLS Session Tickets

## Public Key Infrastructure Considerations

It is important that API server and client implementers consider how their
implementations will interact will Public Key Infrastructure. In
particular implementers SHALL provide a mechanism for deploying and
revoking subject certificates for API servers. API clients SHALL provide
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
enable them to carry out certain actions on APIs. A token is issued to
the client by an authentication server upon successful authentication of
the client. The client can then use that token in requests it sends to
APIs. The token contains *claims* which describe what actions the client
is allowed to do against the API, for example reading or writing to a
particular endpoint. The set of claims a particular token contains is
referred to as being its *scope*. A particular token can contain many
different claims, allowing it to be used for a variety of actions.

Tokens are cryptographically signed, to allow their provenance to be
validated. Various algorithms can be used for this, but RSA with SHA-256
SHOULD be used as the signing algorithm for NMOS APIs to ensure
compatibility.

> TODO: Are we allowing other options, or is this "should" above actually a "shall"?

### Client Authentication

Before tokens can be issued to an API client it SHALL first authenticate
with the authentication server. Clients SHOULD use *OAuth 2.0* to
authenticate. OAuth 2.0 is a widely used authorisation mechanism which
has been used to provide authentication for issuing JSON Web Tokens on
other systems.

![Token acquisition - image courtesy of Riedel<span
data-label="fig:tokenAquisition"></span>](images/nmos_sec_1.png)

The token request and grant process is illustrated above.
This diagram shows a client system
requesting an access token which it could use to, for example, control
IS-05 interfaces on the Node. As the diagram shows, passwords SHALL be
sent to the authentication server. HTTPS SHALL be used; this is absolutely vital
that so that these passwords cannot be intercepted.

Once authentication has been successfully completed, the
authentication server SHOULD confirm that the client has permissions required for
the scope of token it has requested. The authentication server SHALL then issue the token to the
client. Tokens are time limited, and SHALL be renewed with the
authentication server once they expire. This allows permissions to be
revoked by system administrators if required.

The diagram also shows two NMOS Nodes retrieving the
public key from a /certs endpoint. They SHOULD use this to check the
signature on any JSON web tokens they receive, to ensure that they are
genuine. This operation need only be completed once during the lifetime
of the key used by the authentication server to sign the tokens.

The authentication server SHOULD advertise itself using DNS-SD in the same
manner as the IS-04 APIs. Client devices SHOULD automatically
discover the authentication server without manual configuration.

### Client Authorisation

Once a client has obtained a web token from the the authentication
server it SHOULD use that token in API calls it makes. The token SHALL be
placed in the authorisation header of the request, but otherwise the request is the
same as it would normally be when using the API.

As JSON Web Tokens are not encrypted themselves, these requests SHALL be made over HTTPS.
If not encrypted, the Web Token could be intercepted and used to make calls by a third party.

![Client authorisation - image courtesy of Riedel<span
data-label="fig:clientAuthorisation"></span>](images/nmos_sec_2.png)

The process of using the Web Token is illustrated above.
On receiving a Web Token for the
first time the API server SHALL check the validity of the token with the
authentication server’s public key. The API server SHALL then check that
the scope of the web token is sufficient to carry out the operation
requested, and SHOULD only respond as normal if the check is successful.
API servers do not need to contact the authentication
server directly to authenticate the token, provided they have previously
obtained its public key.

## Future Work

In order for this to be implemented the following work is required:

- Definition of claims required for operations on existing AMWA APIs.

- Specification of the mDNS/DNS record advertisements used to
    advertise the authentication server.

> TODO: Is this future work to be performed in a later phase, or is this future work
that needs to be complete prior to publication of this document?
