# OAuth 2.0 for Single Page Applications
If you think that something on this page does not reflect how SPAs work, how OAuth works, 
or if you think that important aspects are missing --- feel free to send a pull request 
to contribute to the discussion!


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
    * via counterfeit resource servers
  * attacker may try to inject code or access token into the authorization response
  

## Comparison of Flows

### OpenID Connect using Implict Flow (response type "token id_token")

 1. Client: create *state* and *nonce* value and store them in browser
 1. Client: use OpenID Connect + response type "token id_token" for authorization request
 1. AS: issue *access token*
 1. AS: issue *ID Token* with sensible sub (transaction specific in case of plain API authorization?)
 1. Client: send CSP (no referrer)
 1. Client: check *state* parameter (CSRF) against state value in browser
 1. Client: check id token:
    1. Client: fetch JWKS file based on issuer URL (requires CORS)
    1. Client: check signature of ID token
    1. Client: check nonce in id token against nonce value in browser
    1. Client: check *at_hash* (https://openid.net/specs/openid-connect-core-1_0.html#ImplicitTokenValidation)
 1. Client: remove URL from browser history
 1. Client: use access token

Security measures needed (1st and only line of defense):

 * ID Token
 * Nonce
 * State
 * Open Redirector is prevented
 * CSP to prevent referer header
 * Browser history manipulation

### Access Token Refresh
Further Authorization Requests are sent to the AS in order to obtain fresh access tokens. Those requests are typically performend in an invisible iFrame. This iFrame is affected by the 3rd party Cookie policy of the browser. 

### Code
 
 1. Client: create *state* value and store it
 1. Client: create *PKCE verifier and challenge* and store verifier
 1. Client: use response type "code" for authorization request
 1. AS: issue *code* linked to client_id and PKCE challenge
 1. Client: check *state* parameter (CSRF) against state value in browser
 1. Client: send *code* along with redirect_uri and PKCE verifier to AS (requires CORS)
 	1. AS: check code expiration/single use
 	1. AS: check code to client_id link
 	1. AS: check code to PKCE challenge link
 	1. AS: issue access token
 1. Client: use access token

Security measures needed (1st line of defense):

 * State
 * PKCE
 
Further Security Measures (2nd line of defense)
 
 * Single use codes
 * Open Redirection is prevented
 * CSP to prevent referer header

### Access Token Refresh
Refreshing via further authorization requests (as described above) is possible, but code also has the ability to issue refresh tokens. The AS may issue a refresh token along with the access token when the client exchanges the code. This refresh token can be used to obtain fresh access tokens using a direct backchannel request between client and AS.

The refresh token themself can be protected against replay in two ways:

 * using dynamically issued secrets (dynamic client registration)
 * using refresh token rotation 

## Observations

Code offers __defense in depth__, OIDC implicit relies on a __single line of defense__.

OIDC implicit relies on the client to implement all security checks. Code places as much as possible security checks at the AS. Given the ratio between clients and AS (many to one), defense is spread across a deployment for OIDC implicit. Moreover, app developers typically are no OAuth & security experts whereas developers of OAuth/OIDC implementations should be more familiar with the specific challenges.

Obtaining fresh access tokens: OIDC implicit requires authorization requests through the browser to obtain fresh access tokens. Such requests, when sent in an hidden iFrame, may fail due to a restrictive 3rd party cookie policy. The reliable solution would be to send those request in a top level window (full page redirect or popup), which would have an negative impact on UX. __This causes clients to use long-lived access tokens, which has an negative impact in case of leakage and replay__. Code supports short-lived and privilege restricted access tokens through refresh tokens: The client can obtain fresh access tokens from the AS's token endpoint at any time without impact on the UX. Short-lived and privilege restricted access tokens reduce the impact of leakage at resource servers. Replay prevention for refresh tokens can be implemented much easier than for access tokens.

OIDC implicit forces the AS (actually the OP) to expose the user's identity to the client (through the *sub* claim) even in cases where the client just want to perform a pure authorization to get access token to an API (e.g. payment initiation). The OP MUST get the consent from the user for this data transfer and the client policy must describe how this data is used and protected (at least under GDPR). The alternative is to provide the client with an ephemeral user id. This in turn requires the AS/OP, for every request with scope "openid", to decide whether to expose a real or a ephemeral user id. That makes the implementation of both the AS and the clients more complex since it can only be controlled by a client-specific fixed policy at the AS.
