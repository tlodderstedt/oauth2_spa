# OAuth 2.0 for Single Page Applications

## Hybrid

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
