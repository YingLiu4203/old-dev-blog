---
layout: post
title: Spring OAuth2 Study Note 
categories:
- Development
tags:
- Java
---

This is a study note for [OAuth2 Getting Started][1] and [Spring OAuth 2 User Guide][2] with examples from [SpringOAuth2 Tutorial][3] and [Spring Boot OAuth2 Tutorial][4]. 

## 1. OAuth2 Introduction
Most OAuth2 APIs follow the OAuth 2.0 draft 10 published in July 2010. There are four roles in the OAuth2 spec.

1. The User: the resource owner. Resources can be any data or services. 
2. The API Server: the resource server that validate access token and grants access based on valid access token. 
3. The Authorizatoin Server: the server that displays the OAuth prompt, approves or denies the access request. It is also responsible for granting access token after the user authorizes the application. Therefore a an authorization server has two primary URLs: one for authorization and one for access token. 
4. The Client: the client is an app acting on the user's behalf to access the user's resources. The client can obtain permssion by either directing the user to the authorization server or by asserting permission direclty with the authorization server without the user interaction. There are two types of clients: confidential client (web apps) and pulbic client (browser code or mobile app). A confidential client has the `client_secret` while a public client doesn't have it. 

There are several types of security data used in the OAuth2 workflow:

1. `client_id` and `client_secret`: the server creates these after an app registers with the server. 
2. Authorization code: the server generates this to let the app exchanges it for an access token. 
3. Access token: the token is used to making authencitated request to the API. A token has expiration time, scope and other information. It can be self-contained or could be a key in a database. 
4. Refresh token: used to get a new access token when an access token expires. 

### 1.1. Authorization Workflow

#### 1.1.1 Registering an App
The first setp in an OAuth process it to register a new app. You give the authorization server app name, web site, logo, and a redirect URI. The server creates a client id and a client secret. The client id is public. The client secret is only kept by a confidential client.

The redirect URI must be an HTTPs endpoints and should be an exact match. Try to avoid query string. OAuth2 provides a "state" parameter to let you store CSRF string or app-specific data. 

#### 1.1.2. Authorization  
There are several types of authrization workflow for differnt apps such as web server apps and broswer/mobile apps. See the grant types for detail workflows. 
 

#### 1.1.3. Making Authenticated Requests
Making a request over HTTPS with the access token like the following: 

```
curl -H "Authorization: Bearer RsT5OjbzRn430zqMLgV3Ia" \
https://api.oauth2server.com/1/me
```

It is also possible to use the token in a post body parameter -- check API server for clarification. 

### 1.2. Access Token  
The access token may have the following properties: 
* access_token 
* token_type: usually "bearer"
* expires_in: duration of time the access token is granted for
* refresh_token: not used for public client requesting implicit grant. 
* scope

The authentication server also sets `Cache-Control: no-store` and `Pragma: no-cache` HTTP headers to ensure clients do not cache this request.

There is standard for the access token structure. Using self-encoded access token to encode all necessary information that is signed by the authorization server. A common technique for this is using the JSON Web Signatrue(JWS) and JSON Web Token(JWT). 

### 1.3. Grant Types 
OAuth2 is flexible by supporting several grant types for different use cases. Following note and diagrams are taken from [OAuthLib document][5]. 

#### 1.3.1. Authorization Code Grant
This is used by confidential clients (Web Server Apps). The client interacts with a user-agent (typically a web browser) and is able to handle redirection from the authorization server.  The workflow is as the following: 


```
+----------+
| Resource |
|   Owner  |
|          |
+----------+
     ^
     |
    (B)
+----|-----+          Client Identifier      +---------------+
|         -+----(A)-- & Redirection URI ---->|               |
|  User-   |                                 | Authorization |
|  Agent  -+----(B)-- User authenticates --->|     Server    |
|          |                                 |               |
|         -+----(C)-- Authorization Code ---<|               |
+-|----|---+                                 +---------------+
  |    |                                         ^      v
 (A)  (C)                                        |      |
  |    |                                         |      |
  ^    v                                         |      |
+---------+                                      |      |
|         |>---(D)-- Authorization Code ---------'      |
|  Client |          & Redirection URI                  |
|         |                                             |
|         |<---(E)----- Access Token -------------------'
+---------+       (w/ Optional Refresh Token)
```

It is often used by a web server app.  The web server app creates a link that points to the authentication server using its `client_id` and redirect uri (must be registered). An example is: 

`https://oauth2server.com/auth?response_type=code&client_id=CLIENT_ID&redirect_uri=REDIRECT_URI&scope=photos` 

The user clicks on the link and sees an authorization page. If the user clicks "Allow", the service redirects the user back to the web server with an authorization code. For example: 

`https://oauth2client.com/cb?code=AUTH_CODE_HERE`

Then the web server app exchanges the auth code for an access token: 

```
POST https://api.oauth2server.com/token
    grant_type=authorization_code&
    code=AUTH_CODE_HERE&
    redirect_uri=REDIRECT_URI&
    client_id=CLIENT_ID&
    client_secret=CLIENT_SECRET
```

The server replies with an access token or an error. 

```
{
    "access_token":"RsT5OjbzRn430zqMLgV3Ia"
}

{
    "error":"invalid_request"
}
```

This is the most secure way to authorize access because the access token and `client_secret` are only visible to the web server apps. 

#### 1.3.2. Implicit Grant
This grant type is typically used by public client (SPA or mobile apps) to obtain access tokens without using authorization code and refresh token. The workflow is as the following: 

```
+----------+
| Resource |
|  Owner   |
|          |
+----------+
     ^
     |
    (B)
+----|-----+          Client Identifier     +---------------+
|         -+----(A)-- & Redirection URI --->|               |
|  User-   |                                | Authorization |
|  Agent  -|----(B)-- User authenticates -->|     Server    |
|          |                                |               |
|          |<---(C)--- Redirection URI ----<|               |
|          |          with Access Token     +---------------+
|          |            in Fragment
|          |                                +---------------+
|          |----(D)--- Redirection URI ---->|   Web-Hosted  |
|          |          without Fragment      |     Client    |
|          |                                |    Resource   |
|     (F)  |<---(E)------- Script ---------<|               |
|          |                                +---------------+
+-|--------+
  |    |
 (A)  (G) Access Token
  |    |
  ^    v
+---------+
|         |
|  Client |
|         |
+---------+
```

The Web-Hosted Client Resource is a an HTML with JavaScript code that is able to extract the access token. 

The `client_secret` is not used in this case. The client has a link: 

`https://oauth2server.com/auth?response_type=token&client_id=CLIENT_ID&redirect_uri=REDIRECT_URI&scope=photos`

When the user clicks the link, an authorization prompt is presented to let the user deny or allow the access. If the user clicks "Allow", the service redirects the user back to the redirect uri with an access token or an error code in the URI fragement. 

`https://oauth2client.com/cb#token=ACCESS_TOKEN`

#### 1.3.3. Resource Owner Password Credential Grant
This grant type is suitable for clients capable of obtaining the user's credentials. It is also used to migrate existing clients from HTTP Basic to OAuth by converting the stored credentials to an access token.  It has the following workflow: 

```
+----------+
| Resource |
|  Owner   |
|          |
+----------+
     v
     |    Resource Owner
    (A) Password Credentials
     |
     v
+---------+                                  +---------------+
|         |>--(B)---- Resource Owner ------->|               |
|         |         Password Credentials     | Authorization |
| Client  |                                  |     Server    |
|         |<--(C)---- Access Token ---------<|               |
|         |    (w/ Optional Refresh Token)   |               |
+---------+                                  +---------------+
```

In this case, the client just posts a request like the following: 

```
POST https://api.oauth2server.com/token
    grant_type=password&
    username=USERNAME&
    password=PASSWORD&
    client_id=CLIENT_ID
```

And the server will reply with an access token in the same format as the other grant type. 

#### 1.3.4. Client Credentials Grant
It's only useb by confidential clients to access the protected resources under its control, or those of another resource owner that have been previously arranged with the authorization server. The workflow is as the following: 

```
+---------+                                  +---------------+
:         :                                  :               :
:         :>-- A - Client Authentication --->: Authorization :
: Client  :                                  :     Server    :
:         :<-- B ---- Access Token ---------<:               :
:         :                                  :               :
+---------+                                  +---------------+
```

## 2. Spring OAuth2

### 2.1. OAuth2 Provider
The provider manages and verifies the OAuth2 tokens used to access the protected resources. In some cases it supplies an interface for a user to grant a client to access the protected resources. 

The provider role is split between Authorization Service and Resource Service. Multiple resource services can share an authorization service. 

The following Spring MVC controller endpoints are required in the Spring Security filter chain to implement authorization service: 
* `AuthorizationEndpoint`: to handle requests for authorization, default URL is `/oauth/authorize`
* `TokenEndpoint`: to handle requests for access token, default URL is `/oauth/token`. 

The following Spring Security request filter is used to implement resource service: 
* `OAuth2AuthenticationProcessingFilter`: to populate the Spring Security context with an `OAuth2Authentication` for the request given an authenticated access token. 

### 2.2. Authorization Server Configuration
The `@EnableAuthorizationServer` annotation, working with any `@Beans` that implements `AuthorizationServerConfigurer`, to configure the authorization server.  Spring creates three separate configurers used by `AuthorizationServerConfigurer`:
* `ClientDetailsServiceConfigurer`: configures an OAuth2 client details such as `clientId`, `secret`, `scope`, `authorizedGrantTypes` and `authorities`. 
* `AuthorizationServerSecurityConfigurer`: defines security constraints on the token endpoints. 
* `AuthorizationServerEndpointsConfigurer`: defines authorization endpoints, grant types, and token service. 

### 2.2.1. Manage Tokens 
The `AuthorizationServerEndpointsConfigurer` has several token-related configuration methods. 

One method is `tokenServices(AuthorizationServerTokenServices tokenServices)` for token services. The `AuthorizationServerTokenServices` interface defines the following services: 

1. `createAccessToken`: create an access token for an OAuth2 authentication. The authentication must be stored so that API server accepting the access token can reference it later. 
2. `refreshAccessToken`: refresh an access token.
3. `getAccessToken`: Retrieve an access token for the provided OAuth2authentication. 

The `DefaultTokenServices` class implements the inteface using random UUID for access token and referesh token. The `DefaultTokenServices` trranslates between token values and authentication information. 

Another method `tokenStore(TokenStore tokenStore)` sets the token store used by the `DefaultTokenServices` to handle token persistence. There are three stores: `InMemoryTokenStore`, `JdbcTokenStore` and `JwtTokenStore`. 

The `JwtTokenStore` doesn't persistent any data. It encode token data using the JSON Web Token (JWT) spec. It depends on a `JwtAccessTokenConverter` that encode and decode the access token. The token are signed by default. To verify an access token, the resource server either needs a shared symmetric key or the public key of the authorization server.  The public key (if available) is exposed by the Authorization Server on the `/oauth/token_key` endpoint, which is secure by default with access rule `denyAll()`. You can open it up by injecting a standard SpEL expression into the `AuthorizationServerSecurityConfigurer`. To use the `JwtTokenStore` you need `spring-security-jwt` on your classpath. 

### 2.2.2 Grant Types
The `AuthorizationServerEndpointsConfigurer` can set the following properties to affect grant types: 

* `authenticationManager`: enables password grant. 
* `authorizationCodeServices`: defines the authorization code services
* `implicitGrantService`: manages state during the imlpicit grant.
* `userDetailsService`: used by refresh token grant

### 2.3. Resource Server Configuration



## Resources: 

1. [Oauth2 Getting Started][1]
2. [Spring Security OAuth2 Document][2]
3. [Spring Secruity OAuth2 Tutorial][3]
4. [Spring Boot Security OAuth2 Totorial][4] 
5. [Python OAuth Lib Grant Types and Workflows][5]

[1]: https://oauth.net/getting-started/
[2]: https://projects.spring.io/spring-security-oauth/docs/oauth2.html
[3]: http://projects.spring.io/spring-security-oauth/docs/tutorial.html
[4]: https://spring.io/guides/tutorials/spring-boot-oauth2/
[5]: http://oauthlib.readthedocs.io/en/latest/oauth2/grants/grants.html