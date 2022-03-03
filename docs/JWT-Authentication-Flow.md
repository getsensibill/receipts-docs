# Bearer Authentication with JWT

## Intro

> **Prerequisites**: familiarity with the [JWT](https://datatracker.ietf.org/doc/html/rfc7519), [JWK](https://datatracker.ietf.org/doc/html/rfc7517), [JWS](https://datatracker.ietf.org/doc/html/rfc7515) and [JWE](https://datatracker.ietf.org/doc/html/rfc7516) specifications.

JWTs can be used with Sensibill API as just another kind of credentials to acquire User or Client Access Tokens but cannot be used as Access Tokens directly. There is only one exception to that - you can use JWTs directly to create a User Account. 

>Sensibill API doesn't support JWTs as Access Tokens themselves except for the User Account Creation. In all other cases, JWTs are exchanged for the actual Access Tokens, so you can consider JWTs with Sensibill as just another kind of credentials that replace API Client Key and Secret (described in the OAuth 2.0 flow).

The diagram below demonstrates how using JWTs for User Registration and Authentication reduces the number of calls that need to be made to the Sensibill API.

![JWT Auth Flow](https://github.com/getsensibill/receipts-docs/raw/main/assets/images/jwt-flow.png 'JWT Auth and Registration Flow')

As you can see from the diagram above only 2 endpoints support JWT - `POST /jwtRegister` and `POST /jwtAuthenticate`. Please continue reading to find out the specifics of the usage for those endpoints.

## JWS Requirements and Format

JWS consists of 3 parts - headers, payload and signature.

### JWS Headers

Only 2 headers are required for JWS to work with Sensibill - `alg` and `kid`.

- `alg`: Sensibill supports **RSA​** or **ECDSA​** algorithms with ​2048-bit​ key size for signatures at the moment. The desired value is to be supplied during the Client Account set up through the Sensibill Support Team.
- `kid`: this is the ID of the public key that should be used for JWT signature verification, it should correspond to one of the IDs returned by the client's public keys URL. See [JWT Signature Verification](./JWT-Authentication-Flow.md#jws-signature-verification) for details. 
- `typ`: this header **is not required** but if present must be equal to `JWT`.

> Desired algorithm for the `alg` header needs to be configured on the Client Account. Please contact the Support Team to do that.

### JWS Payload and Supported Claims

Following claims are supported:
- `sub`: this contains Sensibill User Access ID, which is provided by the client when registering. For the registration endpoint, it will be used to create the user and for the authentication endpoint it will be used to lookup the user. 

> If there is a need to use a claim other than `sub` as the Sensibill User Access ID then it can be configured on the Client Account. Please contact the Support Team.

- `iat`: required and must contain a timestamp of when the JWT was generated. It will be compared against the `max age` parameter which is configured during the Client Account setup. Max age client configuration takes precedence over the `exp` claim described below. It's like a safety net.

> `Max age` of all JWTs is configured on the Client Account. Please contact Support Team for that.

- `exp`: not required but supported and will be respected if present as long as it doesn't exceed the max-age configuration described above.

- `aud`, `iss`, `scp`: not required but supported. To enforce those claims to be checked the desired values must be configured on the Client Account. 

<!-- theme: warning -->
> Please note, `aud`, `iss`, `scp` claims have no power and are not enforced unless the required values are configured on the Client Account. Please contact support to configure.

### JWS Signature Verification

The JWS signature verification is implemented through asymmetric signing algorithms (RSA and ECDSA). That means that Sensibill requires clients to supply public keys to verify the signatures of JWTs signed with a corresponding private key. 
Clients are required to provide a publicly accessible URL with a list of public keys in a JWKS format. That URL will be used by Sensibill to automatically refresh the list of public keys available for signature verification for requests coming in from the client in question. Each JWT is required to have a "kid" header with the id that corresponds to one of the keys in the list returned by that endpoint. 

> `Public Keys URL` needs to be configured on the Client Account. Please contact the Support Team to do that.

At a high level, the verification process looks as described in the following sequence diagram.

![JWT Signature Verification Flow](https://github.com/getsensibill/receipts-docs/raw/main/assets/images/jwt-verification-flow.png 'JWT Signature Verification Flow')

## JWE Support and Requirements

Sensibill API supports encrypted JWTs (JWEs). That is yet another layer of security on top of JWS (please see the specs mentioned in the Introduction) where, apart from signing the JWT, the resulting JWS is also encrypted using Sensibill's public key.

JWE currently supports RSAES OAEP using default parameters for encrypting the Content Encryption Key (CEK) and AES GCM using the 256-bit key for encrypting the content. 

- `alg`: **RSA-OAEP**
- `enc`: **A256GCM**

> Please reach out to the Sensibill Support Team to receive Sensibill’s public key to be used for JWT encryption. JWE’s content is expected to be a JWS conforming to the JWS requirements mentioned earlier.