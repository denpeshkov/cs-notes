---
tags:
  - TODO
  - Security
---

# Overview

OAuth 2.0 is a protocol for authorization.

In the traditional authentication model, application (client) gain access a protected resource on a server by using the resource owner's credentials. This approach has several drawbacks:

- Application need to store the credentials
- Servers are required to support password authentication, despite the security weaknesses inherent in passwords
- Applications gain overly broad access to the resource owner's protected resources
- Resource owners cannot revoke access to an individual application without revoking access to all applications
- Compromise of any application results in compromise of the end-user's password and resources

In OAuth, instead of using the resource owner's credentials, the client obtains an access token. This token is issued by an authorization server with the resource owner's approval

## Roles

- Resource owner - An entity capable of granting access to a protected resource (end-user)
- Resource server - The server hosting the protected resources
- Client - An application making protected resource requests on behalf of the resource owner and with its authorization
- Authorization server - The server issuing access tokens to the client

# References

- [OAuth.com - OAuth 2.0 Simplified](https://www.oauth.com)
- [Demystifying OAuth Flows | Frontegg](https://frontegg.com/blog/oauth-flows)
- [Site Unreachable](https://auth0.com/blog/identity-unlocked-explained-episode-1/)
- [OAuth 2.0 and OpenID Connect (in plain English) - YouTube](https://youtu.be/996OiexHze0?si=RWuqYJmQdK7skJno)
- [An Illustrated Guide to OAuth and OpenID Connect - YouTube](https://youtu.be/t18YB3xDfXI?si=rAIir1R8PvNxz3fM)
- [ID Token and Access Token: What Is the Difference?](https://auth0.com/blog/id-token-access-token-what-is-the-difference/)
- [Everything You Ever Wanted to Know About OAuth and OIDC - YouTube](https://youtu.be/8aCyojTIW6U?si=qYHYU-mv_xbjeCRs)
- [Authentication as a Microservice - YouTube](https://youtu.be/SLc3cTlypwM?si=dRjwHwP5UHdd6wTY)
- [What is OAuth 2.0 and How does it Work?](https://fusionauth.io/articles/oauth/modern-guide-to-oauth)
- [RFC 6749 - The OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749)
- [RFC 6750 - The OAuth 2.0 Authorization Framework: Bearer Token Usage](https://datatracker.ietf.org/doc/html/rfc6750)
