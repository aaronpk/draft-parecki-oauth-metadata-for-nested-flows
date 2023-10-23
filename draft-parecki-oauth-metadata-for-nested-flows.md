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


# Security Considerations

TODO Security


# IANA Considerations

TODO


--- back

# Acknowledgments
{:numbered="false"}

TODO
