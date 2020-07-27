## Grant Types

This blog is the secont part of a blogging series covering Web Security using OAuth 2.0 and OpenID Connect. In the previous part I've introduced you to OAuth 2.0 and OpenID Connect. This blog discusses OAuth 2.0 Grant Types. Enjoy reading!

A Grant Type is how you request and receive an access token



## Authorization Code Flow

Authorization Code Flow, known as the most secure grant type by default. This diagram illustrate 

Authorization code which is a code that the auth server generates when the resource owner authorizes a request.

### Proof Key of Code Exchange

What do we do for SPA or mobile apps where the app can't keep a secret because the compiled source code can be reverse engineered to find the embeded secrets. Proof Key for Code Exchange (PKCE, pronounced pixie), [RFC 7636](https://tools.ietf.org/html/rfc7636#section-1) which is another oAuth 2.0 extension is here to rescue. Instead of having a client secret the client app produces a **code verifier**(kept locally) which is a random code string. Then it generates a **code challenge** which is the code verifier encoded in base64, hashed with SHA-256. When you make an authozired request, the client will send the code challenge and keep the code verifier secret and get the auth code just as before. The client will send the auth code **along with the code verifier** to the auth server to ask for the access token. The auth server will encode the code verifier to base64 and hash it with SHA-256 and compare it against the previously sent code challenge. If they match, the auth server will send back the access token. This way, a malicious attacker can only intercept the Authorization Code, and they cannot exchange it for a token without the Code Verifier. Note that using PKCE you can't get a referesh token because it should not store secrets in the client. 

This process have lots of moving parts and it's error-prone. That's why it's recommended to leverage the OpenID Foundation [AppAuth libraries](https://appauth.io/) for such task. They have great open source libraries for IOS, Android and JavaScript.

## Implicit/Hybrid

## Resource Owner Password

## Client Gredential

## Device Grant Type

## Wrap Up





