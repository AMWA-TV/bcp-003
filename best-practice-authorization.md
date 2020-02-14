# [Work In Progress] Best Practice Authorization

Please see the IS-10 Specification at [NMOS Authorization](https://github.com/AMWA-TV/nmos-authorization)
for further details regarding implementing authorization with the NMOS suite of APIs.

## NMOS API Interactions with IS-10

The IS-10 Specification has outlined the specification for implementing a general OAuth 2.0 authentication system, in
line with current best practices. However the application to an environment that operates other NMOS APIs means
that further security mechanisms can be put into place using the existing APIs.

### IS-04 - Discovery and Registration

When registering resources with an instance of an [IS-04 Registry], the Registry MUST register the Client ID
of the client performing the registration. The Client ID can be found within the `client_id` claim of the
valid JWT access token used in the HTTP request.

Subsequent requests to modify or delete a registered resource MUST validate that the Client ID used to
register the resource matches the `client_id` claim in the JWT access token used in the HTTP request making
the modifications. This is to ensure that clients do not, maliciously or incorrectly, alter resources belonging to
other nodes. Unsuccessful validation of a matching Client ID MUST result in the Resource server rejecting the
request.

[IS-04 Registry]: http://amwa-tv.github.io/nmos-discovery-registration/ "AMWA IS-04 NMOS Discovery and Registration Specification"
