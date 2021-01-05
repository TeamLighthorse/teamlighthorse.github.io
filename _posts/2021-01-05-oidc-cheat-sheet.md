---
layout: post
title: OIDC Cheat Sheet
tags: [oidc, oauth]
excerpt_separator: <!--more-->
author: primarilysoftware
---
It can be hard to wrap your head around OIDC and all its related concepts (flows, scopes, tokens, endpoints, etc.).
This post aims to be a cheat sheet that we can reference when questions or issues arise related to the OIDC flow
that lighthouse uses.
<!--more-->

### What is OIDC
- [Final: OpenID Connect Core 1.0 incorporating errata set 1](https://openid.net/specs/openid-connect-core-1_0.html)
- A layer on top of Oauth 2.0 that allows clients to verify a user's identity who has authenticated with an Authorization Server
- Allows for basic profile information about a user to be shared with client applications

### What is Identity Server?
- [Welcome to IdentityServer4 (latest) — IdentityServer4 1.0.0 documentation](https://identityserver4.readthedocs.io/en/latest/)
- Customizable library and templates that help implement an OIDC Authorization Server in ASP.NET Core

### Key OIDC Endpoints
- Authorization Endpoint
	- https://lighthouse-auth.kelleylogistics.com/connect/authorize
	- In Nucleus, the user is redirected to this endpoint with some query string parameters that tell the identity server which
    client is trying to authenticate the user, and which scopes the client would like the identity server to include in the
    resulting access token
		- Request Parameters
			- client_id=lighthouse-aks
			- redirect_uri=https://lighthouse.kelleylogistics.com/signin-oidc
			- response_type=code id_token (hybrid)
			- scope=profile offline_access permissions manage_addressbook manage_dispatches manage_progression manage_shipments manage_accounting manage_billoflading
			- response_mode=form_post
			- nonce=… (optional, used to prevent replay attacks)
			- state=… (optional, helps prevent CSRF attacks)
	- Identity Server Login Page
		- https://lighthouse-auth.kelleylogistics.com/Account/Login
		- Normal HTML login page.  Form is POSTed, the username and password is validated, and if successful an authentication cookie is set
	- Identity Server Browser Redirect
		- https://lighthouse-auth.kelleylogistics.com/connect/authorize/callback
		- HTML form that automatically posts itself to the client's signin-oidc endpoint.  The form post includes an id_token and authorization_code for the user
	- Client signin-oidc endpoint
		- https://lighthouse.kelleylogistics.com/signin-oidc
		- This is how the browser sends the authorization_code and id_token to the Client application
		- Client will use authorization_code to request an access_token
	- Identity Server Token Endpoint
		- https://lighthouse-auth.kelleylogistics.com/connect/token
		- Client uses this endpoint to exchange the authorization_code for an access_token 
	- Identity Server User Info Endpoint
		- https://lighthouse-auth.kelleylogistics.com/connect/userinfo
		- Client uses this endpoint to returns claims about the authenticated user
		- Client must authenticate with using the access_token returned from token endpoint
		- Sample claims for a user

```json
{
    "sub": "e7aa5e03-69ac-42b5-9214-fbe35bba4e21",
    "ops": "599bf2fb-3678-44d9-a2cd-ff76860ffa78",
    "ops.view": "599bf2fb-3678-44d9-a2cd-ff76860ffa78",
    "acc": "599bf2fb-3678-44d9-a2cd-ff76860ffa78",
    "acc.view": "599bf2fb-3678-44d9-a2cd-ff76860ffa78",
    "adrs": "DFL",
    "cust": "DFL",
    "name": "jguy@kelleylogistics.com"
}
```
