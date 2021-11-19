# JWT Authentication Flow

## Intro

> **Prerequisites**: familiarity with the [JWT](https://datatracker.ietf.org/doc/html/rfc7519), [JWK](https://datatracker.ietf.org/doc/html/rfc7517) and [JWE](https://datatracker.ietf.org/doc/html/rfc7516) specifications.

JWTs can be used with Sensibill API as just another kind of credentials to acquire User or Client Access Tokens but cannot be used as Access Tokens directly. There is only one exception to that - you can use JWTs directly to create a User Account.

>Sensibill API doesn't support JWTs as Access Tokens themselves except for the User Account Creation. In all other cases, JWTs are exchanged for the actual Access Tokens, so you can consider JWTs with Sensibill as just another kind of credentials that replace API Client Key and Secret (described in the OAuth 2.0 flow).

Diagram below demonstrates how using JWTs for User Registration and Authentication reduces the number of calls that need to be made to the Sensibill API.

![JWT Auth Flow](../assets/images/jwt-flow.png 'JWT Auth and Registration Flow')

As you can see from the diagram above only 2 endpoints support JWT - `POST /jwtRegister` and `POST /jwtAuthenticate`. Please continue reading to find out the specifics of the usage for those endpoints.

## JWT Requirements and Format

## JWT Signature Verification

## JWE Support and Requirements