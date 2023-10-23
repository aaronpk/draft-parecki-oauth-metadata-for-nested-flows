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
  RFC9101:
  RFC9126:


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

For the sake of simplicity, we will refer to the parties involved in the flow as:

* User Agent: The browser used by the End-User
* Client: The OAuth client that is being used by the End-User
* Authorization Server (AS): The authorization server the Client interacts with and is expecting to receive tokens from
* Identity Provider (IdP): The authorization server where the End-User has an account and logs in

(In practice, in the inner OAuth flow, the Authorization Server is acting as an OAuth Client to the Identity Provider.)


<artwork type="svg" src="nested-oauth-flow.svg"/>

1. The OAuth Client initiates an OAuth flow by redirecting the User Agent to the Authorization Server.
2. The User Agent visits the Authorization Server.
3. The Authorization Server initiates a new OAuth flow as the OAuth client to the Identity Provider by redirecting the User Agent to the Identity Provider, and provides the additional parameters defined in {{parameters}}.
4. The User Agent visits the Identity Provider.
5. The Identity Provider authenticates the End-User.
6. The Identity Provider issues an authorization code and redirects the User Agent back to the Authorization Server.
7. The User Agent visits the Authorization Server carrying the authorization code from the Identity Provider.
8. The Authorization Server exchanges the authorization code at the Identity Provider for an access token and optional ID token.
9. The Identity Provider responds with the tokens.
10. The Authorization Server consumes the tokens and issues its own authorization code, and redirects the User Agent back to the Client.
11. The User Agent visits the Client carrying the authorization code from the Authorization Server.
12. The Client exchanges the authorization code for an access token at the Authorization Server.
13. The Authorization Server responds with the tokens.

In this example, the two OAuth flows are authorization code flows. In practice, the inner flow can often be the OpenID Connect Implicit Flow (with `response_type=id_token`) where there is no intermediate authorization code issued. This distinction is not relevant to the rest of this specification.

Either or both OAuth flows may also be using other OAuth extensions, such as Pushed Authorization Requests {{RFC9126}}, JWT-Secured Authorization Request {{RFC9101}} or others. While these extensions may change the example sequence above slightly, they do not fundamentally change the need for the additional context added by this specification. See {{use-in-flows}} for examples of how to provide the properties defined in this specification along with other OAuth extensions.


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
  be the actual session cookie. This identifier enables the IdP to identify a user's session.



# Use in OAuth Flows {#use-in-flows}

These parameters can be used in any authorization request to an OAuth Authorization Server or OpenID Provider. Below are examples of including the new parameters in various OAuth flows and extensions.

## OAuth Authorization Request

    https://idp.example.com/authorize?response_type=code
      &state=af0ifjsldkj
      &client_id=s6BhdRkqt3
      &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
      &code_challenge=K2-ltc83acc4h0c9w6ESC_rEMTJ3bww-uCHaoeK1t8U
      &code_challenge_method=S256
      &scope=openid+profile
      &application_class_id=1234
      &application_class_name=Chat+for+iOS
      &device_id=9876
      &device_name=Alice's+iPhone
      &session_ref=5124


## Pushed Authorization Requests (PAR) {#par}

The parameters defined in {{parameters}} are added to the Pushed Authorization Request as POST body parameters, as described in Section 2.1 of {{RFC9126}}.

    POST /par HTTP/1.1
    Host: idp.example.com
    Content-Type: application/x-www-form-urlencoded

    response_type=code
    &state=af0ifjsldkj
    &client_id=s6BhdRkqt3
    &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
    &code_challenge=K2-ltc83acc4h0c9w6ESC_rEMTJ3bww-uCHaoeK1t8U
    &code_challenge_method=S256
    &scope=openid+profile
    &application_class_id=1234
    &application_class_name=Chat+for+iOS
    &device_id=9876
    &device_name=Alice's+iPhone
    &session_ref=5124


## JWT-Secured Authorization Request (JAR) {#jar}

The parameters defined in {{parameters}} are added to the Request Object as described in Section 4 of {{RFC9101}}.

The following is an example of the Claims in a Request Object before the base64url encoding and signing.

    {
     "iss": "s6BhdRkqt3",
     "aud": "https://idp.example.com",
     "response_type": "code",
     "client_id": "s6BhdRkqt3",
     "redirect_uri": "https://client.example.org/cb",
     "scope": "openid profile",
     "state": "af0ifjsldkj",
     "max_age": 86400,
     "application_class_id": "1234",
     "application_class_name": "Chat for iOS",
     "device_id": "9876",
     "device_name": "Alice's iPhone",
     "session_ref": "5124"
    }




# Security Considerations

TODO

## Client Authentication

Because the parameters defined by this specification are not pre-registered at the Identity Provider, the Identity Provider is trusting the Authorization Server with the values provided in the request.

For this reason, the Authorization Server MUST be a confidential client and use some form of client authentication to the Identity Provider.


## Confidentiality

The parameters defined by this specification may contain sensitive data, and should not be exposed to unnecessary parties. In particular, passing this data via the browser in redirects opens up opportunities for observing or tampering with this data.

For this reason, Authorization Servers SHOULD use Pushed Authorization Requests {{RFC9126}} and/or JWT-Secured Authorization Request {{RFC9101}} to prevent tampering and exfiltration of the data.


## Session Reference

The `session_ref` parameter is meant to be a pointer to a session, and MUST NOT be the same value as the browser sends to the server as the session cookie.


# Privacy Considerations

TODO



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

