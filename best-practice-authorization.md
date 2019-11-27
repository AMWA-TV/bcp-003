# [Work In Progress] Best Practice Authorization

## Scope

This document specifies how to implement client authorization for the NMOS APIs.

This is based on best practice used for RESTful APIs, and is intended to promote a secure approach to interoperability.

Use of insecure communication (plain HTTP etc.) is forbidden within the scope of this document.

Implementation of [BCP-003-01](best-practice-secure-comms.md) is a recommended prerequisite to implementing this document.

Although security of web pages presented to users is also important,
this is outside the scope of this document, which is concerned only with APIs.

The mechanism used to authenticate users of a system implementing these recommendations is out of scope.

## Use of Normative Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119][RFC-2119].

## Normative References

These appear at the end of the Markdown source for this document,
and are referenced as hyperlinks within the main body.

## Definitions

_See also the [NMOS Glossary](https://github.com/AMWA-TV/nmos/wiki/Glossary), and definitions within RFCs._

### API

An HTTP / WebSocket API as defined in an AMWA NMOS Specification (IS-04, IS-05, IS-06, etc.)

### Client

The entity that is using the API, for example:

- a Node using the IS-04 Registration API
- a monitoring application using the IS-04 Query API
- a connection control application using the IS-05 Connection API

### Protected Resource

Any part the API that is access restricted. This may apply to read (e.g GET) or write (e.g POST) operations.

### Resource Owner

An entity capable of granting the client access to a protected resource. When the resource owner is a person, it is referred to as an end-user.

### Resource Server

The entity that is providing APIs containing protected resources, for example:

- a Registry implementing IS-04 Registration and Query APIs
- a Node implementing IS-04 Node API and IS-05 Connection API.

### Authorization Server

The server issuing access tokens to the client after successfully
authenticating the resource owner and obtaining authorization.

### Access Token

A short-lived JSON Web Token that may be used by a client to access privileged resources on the resource server.

### Bearer Token

Bearer tokens are longer-lived than access tokens. They are passed from the Authorization Server to the client after successful authentication of client credentials, and contain an access token. Bearer tokens can be used to retrieve further access tokens from the Authorization Server once the original access token expires.

## Introduction (informative)

This document covers client authorization, the mechanism by which an AMWA NMOS API Resource Server may verify that a client accessing
it has the privileges required to access or modify some or all of the content using the API.

This document is not concerned with the security of the connection used to carry out authorization or subsequently authorised interactions, but for the authorization mechanisms described in this document to be effective the connection used must be secured, ideally using the recommendations covered in [BCP-003-01](best-practice-secure-comms.md).

The client authentication mechanism described in this document is based on the OAuth 2.0 Authorization Framework [RFC 6749][RFC-6749]. In particular JSON Web Tokens are used as the OAuth 2.0 Bearer Tokens and for client authorization as per [RFC 7523][RFC-7523].

## Authorization Flow (informative)

A simplified illustration of the authorization flow is shown below. The API client, in this case a broadcast control system, provides the client's credentials to the Authorization Server, which verifies them. The mechanism used to verify client credentials is out of scope for this document, but may involve widely used authentication technologies such as a corporate Single Sign On, Kerberos or Microsoft Active Directory for example.

In its request, the client will also indicate what privileges it wants included in a token. If the Authorization Server concurs that a given client may be permitted the privileges requested, it will grant a Bearer Token containing an Authorization Token whose "claims" include the requested privileges.

The client uses the Authorization Token it has been issued with when it makes requests to protected resources on the Resource Server. The Resource Server then validates the token, using the public key of the Authorization Server. If the Resource Server finds that the token is valid for the protected resource to be accessed, it will allow the API request to proceed.

![Authorization Flow](images/nmos_sec_3.png)

Tokens are signed using a long-lived private key held by the Authorization Server. The Authorization Server makes available its public key to Resource Servers, to allow them to validate tokens using that key.

The Bearer Token issued by the Authorization Server are much shorter-lived than the Authorization Server secret key, but longer-lived than the Access Token. This means that the client may readily employ the access token without needing to ask the end-user for their credentials, but allows system administrators the opportunity to revoke access to the protected resources by the client by refusing to issue a new access token when asked for a renewal.

## Authorization Server Specification

### Authorization Server API

The Authorization Server SHALL present an instance of the AMWA IS-10 NMOS [Authorization API](https://github.com/AMWA-TV/nmos-authorization).

The Authorization Server MAY present multiple versions of the API on the same port, but MUST name-space them accordingly as per the API specification.

The Authorization Server must otherwise be implemented as per [RFC 6749][RFC-6749].

### DNS-SD Advertisement

The Authorization Server MUST support advertising itself using unicast DNS-SD as per [RFC 6763][RFC-6763].
The Authorization Server SHOULD NOT be advertised via mDNS-based DNS-SD.
The Authorization Server MUST advertise itself with the following service type:

```
_nmos-auth._tcp
```

The hostname and port of the Authorization Server MUST be identified via the DNS-SD advertisement, with the full HTTPS path then being resolved via the standard NMOS API path documentation. A DNS A record MUST be provided to allow the hostname to be resolved. The hostname and domain used MUST match one of the Subject Alternate Names provided in the TLS certificate of the Authorization Server. The hostname and domain used SHOULD be the Common Name on the TLS certificate of the Authorization Server.

Multiple DNS-SD advertisements for the same API are permitted where the API is exposed via multiple ports and/or protocols.

Clients and Resource Servers MUST support discovering the Authorization Server through use of unicast DNS-SD service discovery, as described in [RFC 6763][RFC-6763].

Clients and Resource Servers MUST verify the TLS certificate of the Authorization Server. Clients MUST check that the address of the Authorization Server matches either a Subject Alternate Name or Common Name on the TLS certificate.
Clients MUST verify the entire chain of trust of the Authorization Server TLS certificate, back to a trusted root certificate.
This ensures that the authorization server is trusted.
Clients and Resource Servers MUST provide a mechanism for installing root CA certificates used for verifying the TLS certificate of Authorization Servers. HTTPS MUST be used for all connections to the Authorization Server. Clients and Resource Servers MUST NOT interact with Authorization Servers if the TLS certificate of the Authorization Server cannot be validated.

#### DNS-SD TXT Records

##### api_ver

The DNS-SD advertisement MUST be accompanied by a TXT record of name 'api_ver'. The value of this TXT record is a comma-separated list of API versions supported by the server. For example: 'v1.0,v1.1,v2.0'. There should be no whitespace between commas, and versions should be listed in ascending order.

##### pri

The DNS-SD advertisement MUST include a TXT record with key 'pri' and an integer value. Servers MAY additionally present a matching priority via the DNS-SD SRV record 'priority' and 'weight' as defined in [RFC 2782][RFC-2782]. The TXT record should be used in favour of the SRV priority and weight where these values differ, in order to overcome issues in the Bonjour and Avahi implementations. Values 0 to 99 correspond to an active NMOS Authorization Server API (zero being the highest priority). Values 100+ are reserved for development work to avoid colliding with a live system.

### Authorization Server Public Key

The Authorization Server MUST provide all public keys used for signing tokens at the `certs` endpoint of the API. The Authorization Server MAY present more than one key on this endpoint, with each key being an entry in an array.

All public keys must be presented using the text representation used by The Secure Shell (SSH) Public Key File Format in [RFC 4716][RFC-4716]. Each public key presented will be one entry in the array provided by the `certs` endpoint.

Resource Servers SHOULD seek to fetch public keys from the Authorization Server at least once every hour.
Resource Servers MUST vary their retrieval interval at random by up to at least one minute to avoid overloading the Authorization Server due to Resource
Servers synchronising their retrieval time.
If a Resource Server is unable to contact the Authorization Server, the Resource Server MUST implement a random back-off mechanism to avoid overloading the Authorization Server in the event of a system restart.
Also if a Resource Server is unable to contact an Authorization Server, the Resource Server MAY assume currently held public keys remain valid until it is able to re-establish a connection to an Authorization Server.

Resource Servers SHOULD attempt to verify tokens against every public key presented at its Authorization Server's `certs` endpoint, until the Resource Server finds a public key that verifies the token, or until no keys are left. If a client fails to verify all public keys available, the client MUST reject the token.

#### Changing Keys

When transitioning to a new public/private key pair used for signing tokens the Authorization Server SHOULD provide both the old and new public key at the `certs` endpoint until all tokens that may be verified by the old public key would have expired. However, if a private key is known to be compromised, the Authorization Server MUST remove it from the `certs` endpoint immediately.

Authorization Servers SHOULD provide new public keys on the `certs` endpoint for at least 2 hours before issuing tokens signed by the corresponding private key to allow time for clients to cache the new public key.

## Client Registration

Clients MUST be registered with the Authorization Server before initiating the OAuth 2.0 protocol as per
Section 2 of [RFC 6749][RFC-6749].
The Authorization Server MUST NOT accept unregistered clients.

Authorization Servers SHOULD support a manual mechanism for registering clients (e.g an HTML web form)
allowing the client type and redirect URIs to be provided. Clients SHOULD support a mechanism for providing
the information required for OAuth 2.0 registration (e.g a web page containing the required details).

Clients operating with a client password SHOULD support using HTTP Basic Authentication, as per Section 2
of [RFC 2617][RFC-2617], to authenticate with the Authorization Server
in the manner described in Section 2.3.1 of [RFC 6749][RFC-6749]. Authorization Servers SHOULD provide
support for such registrations.

Clients and Authorization Servers are RECOMMENDED to provide for the OAuth 2.0 Dynamic
Client Registration Protocol [RFC 7591][RFC-7591].

AMWA NMOS Specifications MAY specify additional registration parameters where they require them.
Where clients and Authorization Servers that support OAuth 2.0 are used with these specifications
they SHALL use these parameters in all OAuth 2.0 registration mechanisms they support.

## OAuth Grants

### Client Types (Informative)

[RFC 6749][RFC-6749] defines three different classes of OAuth client:
- Web application - client credentials stored in a server.
- User-agent-based application - client credentials stored in the user-agent (e.g browser).
- Native application - client credentials stored in a native application (e.g broadcast control system).

It is important to understand what kind of client is being implemented, as this impacts on the OAuth 2.0
grant that it may use.

Typically clients for NMOS APIs are Broadcast Control Systems, or the broadcast equipment they control
(a "Node" in the [JT-NM Reference Architecture](http://jt-nm.org/RA-1.0/)).

Clearly a broadcast control system that is built as a native app is a "Native Application" type, and
a control system implemented in a browser is a "User-agent-based application", and they should be treated
accordingly when implementing OAuth. [RFC 6749][RFC-6749] defines both these client types to be
_Public Clients_.

Out of these three client types an NMOS Node most closely resembles a web application, because client
credentials are not stored in the user-agent or a native application.
Instead they are stored on a server away from the resource owner.
The web application client type is the only OAuth 2.0 client type where this is
permitted to be the case. [RFC 6749][RFC-6749] considers such clients to be _Confidential Clients_.

### Grant Types

OAuth 2.0 defines four different grant types:

- Authorization Code Grant
- Implicit Grant
- Resource Owner Password Credentials Grant
- Client Credentials Grant.

The authorization code grant is optimised for confidential clients, and is a good choice for NMOS Node type
clients devices, but is less well suited to control system type clients.

The implicit grant is designed for public clients, and therefore is unsuited to confidential clients, but is
generally used for use-agent based clients.

The resource owner password grant is designed for situations where the resource owner has a strong trust
relationship with the client - this is typically reserved for clients like operating systems or other highly
privileged software. This may be the case for native application broadcast control systems, but not those
implemented in the user-agent (e.g browser). This grant is not suitable for NMOS Node type clients.

The client authentication grant is suitable for confidential clients, but requires the resource owner
arrange for the Authorization Server to allow access to protected resources out of band rather than as part
of the grant process.
This may be suitable for use with NMOS, especially where the Authorization Server forms part of a
larger broadcast control system.

Individual AMWA NMOS specifications MAY specify the grants permitted for the API clients involved in the
specification, however the guidance above for the suitability of different grants for user with certain
clients SHOULD be followed. Unless otherwise specified by the AMWA NMOS Specifications it supports, an
Authorization Server SHOULD support all four grant flows.

## Tokens

### Authorization Server Response

Successful authorization requests shall be serviced by the Authorization Server as defined in
[RFC 6749][RFC-6749] Section 5.1.
Additionally the `expires_in` and `refresh_token` fields MUST be included in the response.

Unsuccessful authorization responses should be handled as per Section 5.2 of [RFC 6749][RFC-6749].

### Access Tokens

The access token type returned MUST be of the `Bearer_token` type specified in [RFC 6750][RFC-6750].

The access token MUST be a JSON Web Signature (JWS) as defined by [RFC 7515][RFC-7515].
JSON Web Algorithms (JWA) MUST NOT be used.

The JWS MUST be signed with `RSASSA-PKCS1-v1_5 using SHA-512`, meaning the value of the `alg` field in the
token's JOSE (JSON Object Signing and Encryption) header (see [RFC 7515][RFC-7515]) MUST be set to `RS512`
as defined in [RFC 7518][RFC-7518]. An example JOSE header would be:

```json
{
  "typ": "JWT",
  "alg": "RS512"
}
```

#### Registered Claims

Registered claims are defined in the authorization JSON Web Token specification in [RFC 7519][RFC-7519], but their specific usage is left to the application. NMOS API Clients and Servers implementing this BCP MUST employ the restrictions on claims outlined below, in addition to implementing tokens as specified in [RFC 7519][RFC-7519].

##### iss
_Identifies principal that issued the JWT_

The `iss` (issuer) claim MUST be included in the token. The claim MUST contain the DNS
name and port of the Authorization Server accessible to the audience of the token.
The contents of this claim MUST match one entry in the common name field or alternate
names fields in all TLS certificates used by the Authorization Server for securing the authorization
API, in order to allow the verifier of the token to fetch the public key of the Authorization Server.

##### sub
_Identifies the subject of the JWT_

The `sub` (subject) claim MUST be included in the token. This claim MUST contain a unique identifier assigned
to the client by the Authorization Server. For example, this may be a username in the system, or an
email address of the user. This is primarily intended for audit use (e.g to appear in logs), and as such
should be meaningful in that context.

##### aud
_Identifies the recipients of the JWT_

The `aud` (audience) claim MAY be included in the token, but MUST be included where an AMWA NMOS
Specification with which the token is to be used requires it.

AMWA NMOS APIs are free to specify their own audience claims, but in the absence of specification resource
servers SHOULD assume that the audience claim contains the fully resolved domain name of the intended
recipient. If the `aud` claim is present and does not match the fully resolved domain name of the resource
server, the Resource Server MUST reject the token.

##### exp
_Expiration time of the token_

The `exp` (expiration) claim MUST be included in the token. This is defined in [RFC 7519][RFC-7519] as being a JSON
NumericDate field, which uses the UTC epoch. This is in contrast to the TAI epoch used elsewhere
within the NMOS APIs, so implementers should take care to ensure they are using the correct
epoch.
API implementations MUST reject a token where the `exp` claim value is less than the current UTC
time.

##### iat
_Token issued at time_

The `iat` (issued at) claim MAY be included in the token. As with the `exp` claim this claim uses UTC. API
implementations MUST reject a token where the `iat` claim is greater than the current UTC time.
Authorization Servers SHOULD NOT issue a token with an `iat` claim that is significantly greater or
less than the UTC time at which the token is issued.

#### Private Claims

[RFC 7519][RFC-7519] allows for "private claims". The following claim is used to identify the API
specification a given token is used for.

##### x-nmos-api
_Contains information particular to the NMOS API the token is intended for_

The `x-nmos-api` claim MUST be included in the token. The value of the claim is a JSON object, the
contents of which are defined by AMWA specifications.
The only entry in this object required by this specification is the "name" field. This should be the
identifier of the AMWA specification the token is to be used for in lower case. For example the IS-04
minimal claim is as follows:

```json
"x-nmos-api": {
  "name": "is-04"
}
```
In addition, specifications MAY require a `version` field be included, to indicate which version of the API
the token may be used with. This is to allow the Authorization Server to return an Authorization Token compatible
with the requirements for a particular version of the API.
This field MUST contain a JSON array. Each element of the array MUST be a version number of the API
(as a string formatted as v<#MAJOR>.<#MINOR>) with which the token may be used.

```json
"x-nmos-api": {
  "name": "is-04",
  "version": ["v1.0","v1.1","v1.2"]
}
```

In addition individual AMWA specifications MAY also specify additional entries in the object.
For example, IS-04 could define an entry "node-read", which may be either `true` or `false`.
An example of the resulting `x-nmos-api` claim  is shown below.

```json
"x-nmos-api": {
  "name": "is-04",
  "version": ["v1.0","v1.1","v1.2"],
  "node-read": true
}
```

Authorization Servers MAY only have support for certain NMOS API specifications, and MAY only
support certain versions of such APIs.

#### Size Considerations

While [RFC 7519][RFC-7519] does not prescribe a maximum size for an OAuth 2.0 JSON
Web Token, it should be noted that these tokens are typically used within an HTTP header.
While HTTP does not define a header size limit, 8KByte is a common limitation in HTTP
server implementations.
Specification writers should be mindful of this when designing API claims, and ensure that enough space is
available in the header for both the token and all other HTTP headers.

#### Example Authorization Token Claim Set

```json
{
  "iss": "https://auth.example.com",
  "sub": "username@example.com",
  "aud": "https://node.example.com",
  "iat": "1548779460",
  "exp": "1548783060",
  "x-nmos-api": {
    "name": "is-04",
    "node-read": true
  }
}
```

### Token Lifetime and Refresh

A given token for an NMOS API may be used on more than one Node or Registry instance.
If one of these Nodes or Registries is compromised, it is possible for that entity to "capture" that token,
and use it itself maliciously.
While the precise duration of token validity should be left to implementers and administrators based on the
risk profile, these tokens SHOULD ideally be set to be valid for no more than one hour.

However, if tokens are too short-lived, the number of refresh requests to the Authorization Server for new
tokens starts to become a significant overhead, and any latency involved in using a token may cause it to
become invalid during transit. As such it is RECOMMENDED that tokens be valid for at least 30 seconds.
Clients SHOULD ensure tokens are refreshed sufficiently in advance of their expiry.
While the exact time will depend on the implementation of the client and Authorization Server, it is
RECOMMENDED to attempt a refresh at least 15 seconds before expiry
(i.e the half-life of the shortest-lived token possible).

Authorization Servers MUST issue refresh tokens in the Bearer token returned after the completion of an
authentication flow. Refresh tokens MAY be valid indefinitely or MAY be time-limited.

### Accessing Protected Resources

When accessing protected resources clients MUST include the authorization token in the
request using the Authorization Request Header Field method described in Section 2.1
of [RFC 6750][RFC-6750]. Clients MUST NOT use any of the other methods specified in Section 2.0
of [RFC 6750][RFC-6750].

When a protected resources receives a token it MUST validate the claims of the token.
If a token is invalid the resource server MUST reject the request with the appropriate
HTTP error code as defined by [RFC 6750][RFC-6750].

### Operation with WebSockets

Where OAuth 2.0 is to be used with WebSockets clients SHALL provide the access token
in the HTTP GET request that initiates the Websocket handshake defined in
[RFC 6455][RFC-6455] in the same manner as a normal HTTP request as described in
[Accessing Protected Resources](#accessing-protected-resources).

The Resource Server SHALL validate such tokens in the same manner as it would for a normal
protected HTTP resources using the HTTP Authorization Request Header.

Further to this, due to limitations in the native JavaScript WebSocket API, clients MAY
pass the OAuth 2.0 access token in the query parameters of the request URL during the handshake,
using the `access_token` key, as described in [RFC 6750](https://tools.ietf.org/html/rfc6750#section-2.3).
This is only for situations in which it is not feasible to pass the token in the HTTP Authorization Header.

```
https://www.example.com/ws?access_token=mF_9.B5f-4.1JqM
```

The Resource Server MUST support the parsing of access tokens in both the Authorization
HTTP Header and the query parameter. The Resource Server SHALL NOT upgrade the connection to a
WebSocket if the token is deemed invalid.

## Interaction With Other AMWA Specifications

### AMWA NMOS IS-04

TODO: Needs further discussion with IS-04 group

[//]: ## (References)

[//]: ### (Normative)

[RFC-2119]: https://tools.ietf.org/html/rfc2119 "Key words for use in RFCs to Indicate Requirement Levels"

[RFC-2617]: https://tools.ietf.org/html/rfc2617 "HTTP Authentication: Basic and Digest Access Authentication"

[RFC-2782]: https://tools.ietf.org/html/rfc2782 "A DNS RR for specifying the location of services (DNS SRV)"

[RFC-4716]: https://tools.ietf.org/html/rfc4716 "The Secure Shell (SSH) Public Key File Format"

[RFC-6455]: https://tools.ietf.org/html/rfc6455 "The WebSocket Protocol"

[RFC-6749]: https://tools.ietf.org/html/rfc6749 "The OAuth 2.0 Authorization Framework"

[RFC-6750]: https://tools.ietf.org/html/rfc6750 "The OAuth 2.0 Authorization Framework: Bearer Token Usage"

[RFC-6763]: https://tools.ietf.org/html/rfc6763 "DNS-Based Service Discovery"

[RFC-7515]: https://tools.ietf.org/html/rfc7515 "JSON Web Signature (JWS)"

[RFC-7519]: https://tools.ietf.org/html/rfc7519 "JSON Web Token (JWT)"

[RFC-7523]: https://tools.ietf.org/html/rfc7523 "JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants"

[RFC-7591]: https://tools.ietf.org/html/rfc7591 "OAuth 2.0 Dynamic Client Registration Protocol"
