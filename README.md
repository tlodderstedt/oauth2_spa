# OAuth 2.0 for Single Page Applications
```diff
--- If you think that something on this page does not reflect how SPAs work, how OAuth works, 
--- or if you think that important aspects are missing --- feel free to send a pull request 
--- to contribute to the discussion!
```

## SPAs
We assume a "real" SPA, i.e., the web server of the application only serves static files.

## Threat Model

For the purpose of this document and the related discussion, we focus on **authorization** (i.e., an attacker is not able to use/access the resources of a user, neither through the Client nor directly). (Session Integrity can be discussed separately.)

We assume that...

  * the contents of the authorization response can leak (at least) in the following ways:
    * via the Referer header (e.g., third-party resources in the SPA),
    * via (proxy) server logs 
    * via the browser history
    * via XSS
  

## Comparison of Flows

### Hybrid

 1. Client: create *state* and *nonce* value and store them in browser
 1. Client: use OpenID Connect + response type "token id_token" for authorization request
 1. AS: issue *access token*
 1. AS: issue *ID Token* with sensible sub (transaction specific in case of plain API authorization?)
 1. Client: send CSP (no referrer)
 1. Client: check *state* (CSRF)
 1. Client: check id token:
    1. Client: fetch JWKS file based on issuer URL
    1. Client: check signature of ID token
    1. Client: check nonce in id token
    1. Client: check *at_hash* (https://openid.net/specs/openid-connect-core-1_0.html#ImplicitTokenValidation)
 1. Client: remove URL from browser history
 1. Client: use access token

Security measures needed:

 * ID Token with Nonce
 * State
 * Open Redirector is prevented
 * CSP to prevent referer header
 * Browser history manipulation

### Code
 
 1. Client: create *state* value and store it
 1. Client: create *PKCE verifier and challenge*
 1. Client: use response type "code" for authorization request
 1. AS: issue *code* linked to client_id and PKCE challenge
 1. Client: check *state* (CSRF)
 1. Client: send *code* along with redirect_uri and PKCE verifier to AS
 1. AS: check code expiration/single use
 1. AS: check code to client_id link
 1. AS: check code to PKCE challenge link
 1. AS: issue access token
 1. Client: use access token

Security measures needed:

 * State
 * PKCE
