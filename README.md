# OAuth 2.0 for Single Page Applications
```diff
--- If you think that something on this page does not reflect how SPAs work, how OAuth works, 
--- or if you think that important aspects are missing --- feel free to send a pull request 
--- to contribute to the discussion!
```

## SPAs
We assume a "real" SPA, i.e., the web server of the application only serves static files.

## Threat Model
We assume that...

  * the contents of the authorization response can leak (at least) in the following ways:
    * via the Referer header (e.g., third-party resources in the SPA),
    * via (proxy) server logs 
    * via the browser history
    * via XSS
  * client authentication is used
  
## Security Goals
For the purpose of this document and the related discussion, we focus on **authorization** (i.e., an attacker is not able to use/access the resources of a user, neither through the Client nor directly). (Session Integrity can be discussed separately.)

## Attacks and Mitigations

The ultimate goal of an attacker is to get hold of an access token. If the attacker has learned the access token, he can send any requests to a resource server which the JS SPA could send itself.

### Attack on Implicit

  - An attacker can get hold of an access tokens when they leak from the authz response.
    * What is a mitigation against this? -> ID token is not.
  
### Attack on Authorization Code Flow
  - Access tokens are contained only in the state of the SPA. The attacker cannot directly access this state.
  - Auth codes can leak from the authz response. An attacker who learns an authorization code could try to 
    * exchange it for an access token himself, which he will not be able to do, since he does not know the necessary PKCE verifier; or
    * inject it into a token request sent by **his own** SPA instance in his own browser, which will not use the correct PKCE verifier either.

### Attacks that do not work 
  * There is no point in an attacker trying to inject an access token: 
     * If he tries to do this on a user's machine, he will break session integrity, not authorization. 
     * If he tries to do this on his own machine, he will not be able to do anything more than he could do without injecting it, since he knows and controls the SPA's state anyway.

## Secure Flows

## Hybrid (this is not secure yet!)

  * Client: create state value and link it to local state
  * Client: use OpenID Connect + response type "token id_token" for authorization request
  * AS: issue access token
  * AS: issue ID Token with sensible sub (transaction specific in case of plain API authorization?)
  * Client: check state (CSRF)
  * Client: fetch JWKS file based on issuer URL
  * Client: check signature 
  * Client: check at_hash (https://openid.net/specs/openid-connect-core-1_0.html#ImplicitTokenValidation)
  * Client: CSP (no referrer)
  * Client: prevent open redirection
  * Client: remove URL from browser history
  * Client: use access token

## Code

  * Client: create state value and link it to local state
  * Client: create PKCE verifier and challenge (S256 only!!)
  * Client: use response type "code"
  * AS: issue code linked to client_id and PKCE challenge
  * Client: check state (CSRF)
  * Client: send code along with redirect_uri and PKCE verifier to AS
  * AS: check code expiration/single use
  * AS: check code to client_id link
  * AS: check code to PKCE challenge link
  * AS: issue access token
  * Client: use access token
