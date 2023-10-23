---
title: "OAuth Client and Device Metadata for Nested Flows"
category: std

docname: draft-parecki-oauth-metadata-for-nested-flows-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Web Authorization Protocol"
keyword:
 - oauth
 - delegation
venue:
  group: "Web Authorization Protocol"
  type: "Working Group"
  mail: "oauth@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/oauth/"
  github: "aaronpk/oauth-metadata-for-nested-flows"
  latest: "https://aaronpk.github.io/oauth-metadata-for-nested-flows/draft-parecki-oauth-metadata-for-nested-flows.html"

author:
 -
    fullname: Aaron Parecki
    organization: Okta
    email: aaron@parecki.com
    uri: https://aaronparecki.com

normative:
  RFC6749:
  OpenID:
    title: OpenID Connect Core 1.0
    target: https://openid.net/specs/openid-connect-core-1_0.html
    date: November 8, 2014
    author:
      - ins: N. Sakimura
      - ins: J. Bradley
      - ins: M. Jones
      - ins: B. de Medeiros
      - ins: C. Mortimore


informative:


--- abstract

This specification defines a vocabulary and method of transmitting
information about an OAuth client when a user is redirected through one
or more authorization servers during an OAuth flow. This
provides the deeper nested authorization servers with additional context
that they can use for informational or revocation purposes.


--- middle

# Introduction

In a typical OAuth flow, the OAuth client redirects the user agent to the
authorization server where the user logs in and (optionally) approves the
request. The OAuth framework describes the interaction between the Client,
the Authorization Server, and the Resource Server. The interaction between
the End-User and the Authorization Server, when the user logs in,
is intentionally out of scope of OAuth. In practice, user authentication at
the Authorization Server happens via a wide variety of methods, including
simple username/password login, as well as redirecting to additional OAuth
or OpenID Connect servers, such as when using consumer social login providers
or enterprise identity providers.

When user authentication happens via an external identity provider, it
takes place as a brand new OAuth flow from the initial authorization server to the
external identity provider. The initial authorizaiton server acts as an OAuth
client to the external identity provider. Because this is a new flow, the context of the
original OAuth flow is lost, and the external identity provider is unable
to take actions or record information based on the actual OAuth client the
End-User is using.

This specification introduces a vocabulary and method of transmitting
information about an OAuth client to an external identity provider when
used in nested flows.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

## Terminology

This specification uses the terms "Access Token", "Authorization Code",
"Authorization Endpoint", "Authorization Server" (AS), "Client", "Client Authentication",
"Client Identifier", "Client Secret", "End-User", "Grant Type", "Protected Resource",
"Redirection URI", "Refresh Token", "Resource Owner", "Resource Server" (RS)
and "Token Endpoint" defined by {{RFC6749}},
and the terms "OpenID Provider" (OP) and "ID Token" defined by {{OpenID}}.

TODO: Replace RFC6749 references with OAuth 2.1

This specification defines the following terms:

"Application Class":
: The type of application the End-User is logging in to, as defined by the AS
  in the first OAuth flow.

"Device":
: The physical device the End-User is interacting with when authorizing a Client.

This specification uses the term "Identity Provider" (IdP) to refer to
the Authorization Server or OpenID Provider that is used for End-User authentication.


# Protocol Overview {#overview}



1. The OAuth Client initiates an OAuth flow with the Authorization Server.
1. The Authorization Server initiates an OAuth flow with the Identity Provider.
1. The Identity Provider authenticates the End-User and redirects them back to the Authorization Server.
1. The Authorization Server redirects the End-User back to the Client.
1. The Client finishes the authorization flow by obtainining tokens from the Authorization Server.


# Parameters {#parameters}

This specification introduces new parameters in the authorization request to the Identity Provider
to indicate information about the downstream OAuth client:

"application_class_id":
: An identifier for the Application Class, unique within the Authorization Server

"application_class_name":
: A human-readable name describing the Application Class, e.g. "Chat App for iOS"

"device_id":
: An identifier for the Device the End-User is using

"device_name":
: A human-readable name describing the End-User's device, e.g. "Alice's iPhone"

"session_ref":
: A reference to the End-User session at the OAuth Authorization Server. This MUST NOT
  be the actual session ID. This identifier enables the IdP to identify a session.



# Use in OAuth Flows

These parameters can be used in any authorization request to an OAuth Authorization Server or OpenID Provider. Below are examples of including the new parameters in various OAuth flows and extensions.

## OAuth Authorization Request

```
https://idp.example.com/authorize?response_type=code
  &client_id=CLIENT_ID
  &scope=openid+profile
  &application_class_id=1234
  &application_class_name=Chat+for+iOS
  &device_id=9876
  &device_name=Alice's+iPhone
  &session_ref=5124
```


## Pushed Authorization Requests (PAR) {#par}

```
POST https://idp.example.com/par

client_id=CLIENT_ID
&scope=openid+profile
&application_class_id=1234
&application_class_name=Chat+for+iOS
&device_id=9876
&device_name=Alice's+iPhone
&session_ref=5124
```


## Rich Authorization Request (RAR) {#rar}

```
TBD
```

## JWT-Secured Authorization Request (JAR) {#jar}

```
TBD
```




# Security Considerations

TODO Security


# IANA Considerations {#iana}

TODO


--- back

# Document History

(( To be removed from the final specification ))

-00

* Initial Draft


# Acknowledgments
{:numbered="false"}

TODO

