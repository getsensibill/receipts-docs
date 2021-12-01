# OAuth2 flow

## Intro

> **Prerequisites**: familiarity with the OAuth 2.0 flow in general (see [OAuth 2.0 documentation](https://oauth.net/2/))

As mentioned in the [Introduction](./Introduction.md), you will need a Sensibill Client Account first. It will allow you to create User Accounts for each of the end-user within the client-side app.

Sensibill API supports OAuth 2.0 flow for acquiring client-level access to the API as well as user-level access. This is achieved through **Client and User Access Tokens**.

> To configure your Client Account with the OAuth 2.0 auth, please create a request with the Sensibill Support Team. Once configured the Support Team will provide you with **Client Key and Secret**, **credential type** and the **redirect URL** (see below for usage).

## Flow overview

Client credentials (**API Key and Secret**) allow you to create a user account. 

A User Account is created by providing the desired **User Access Id** (a.k.a username) and **User Access Secret** (a.k.a. password). 

The idea behind utilizing the OAuth 2.0 flow with Sensibill API was that it allows you the flexibility of either allowing users to pick their username and password or generate the username and password for them behind the scenes and authenticate them transparently behind the scenes in a **Single Sign-On** (**SSO**) kind of approach. 

<!--theme: warning-->
> In case you are embedding Sensibill functionality into an existing app or using Sensibill API directly (*the overwhelming majority of the cases*) you should follow the SSO approach and generate the **User Access Id** and **User Access Secret** for your users in the **Integration Server** rather than ask them to provide it.

Once the **User Account** is created you can then acquire the **User Access Token** following the Authorization Code Flow (see [Authorization Code](https://datatracker.ietf.org/doc/html/rfc6749#section-1.3.1) specs).

Considering the SSO approach mentioned above here is how the OAuth 2.0 flow looks like with Sensibill.

![Oauth 2.0 Flow with Sensibill](../assets/images/oauth2-flow.png 'Oauth 2.0 Flow with Sensibill')

Let's review all the steps in detail.

## Acquiring Client Access Tokens

**Client Access Tokens** are used to perform user management, i.e. creating, deleting or disabling user accounts. 

To acquire a Client Access Token, you will need to use the `POST /accessToken` endpoint and supply the **Client Key and Secret** provided to you during the Client Account setup. If you don't have it please contact the Sensibill Support Team.

You will also need to supply the grant type as `client_credentials` (see the [Client Credentials](https://oauth.net/2/grant-types/client-credentials/) specs).

The Client Key and Secret are supplied as a [Basic Auth header](https://en.wikipedia.org/wiki/Basic_access_authentication).

<!--
title: "Requesting Client Access Token using Basic Auth header"
-->
```curl
curl --request POST 'https://receipts-sandbox.sensibill.io/api/v1/accessToken' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic dGVzdEFwaUtleTp0ZXN0QXBpU2VjcmV0' \
--data-raw '{
	"grant_type": "client_credentials"
}'
```

Where `dGVzdEFwaUtleTp0ZXN0QXBpU2VjcmV0` is the base64 encoded API Key and Secret separated by a semicolon `:` (like so `base64encode(clientKey:clientSecret)`).

## Registering User Accounts

Once the Client Access Token is acquired you can now create a User Account using the `POST /users` endpoint. The Client Access Token should be submitted in the Authorization Header as a Bearer token.

The only required parameters in the body are `credentialType`, `accessID` and `accessSecret`.

You should have received the `credentialType` from the Sensibill Support team together with the rest of the credentials when the Client Account was created.

As for `accessID` and `accessSecret`, as discussed above, those should be generated on the backend (in the Integration Server) to make the integration completely transparent for the end-user. 

The rest of the fields are optional for this endpoint. Please refer to the API reference for details.

<!--
title: "Registering new User Account with generated User Access ID and Secret"
-->
```curl
curl --request POST 'https://receipts-sandbox.sensibill.io/api/v1/users' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer AZjaZzze0koUawD4wEg9uatbEIzodERHfNybys8REDiTevwZKZeYg9VejLyioLpdBiDy4cVAfueMF4v0pg9Q' \
--data-raw '{
    "credentialType": "some_bank",
    "accessID": "someUsername",
    "accessSecret": "somePassword",
}'
```

In response to the above request, you should receive a User Id which is a system-generated globally unique ID of the created user in the Sensibill's system.

## Acquiring User Access Tokens

**User Access Tokens** are required to access the document management API endpoints.

Now that the User Account is created you can request a User Access Token for it to start using the document management API on behalf of that user.

Following the [Authorization Code Flow](https://datatracker.ietf.org/doc/html/rfc6749#section-1.3.1) you will need to first acquire the authorization code. 

### Getting Authorization Code for the User Account

To do that you will need to use `GET /authorizationGrant` endpoint and supply the generated User Access ID and Secret in the [Basic Auth Authorization header](https://en.wikipedia.org/wiki/Basic_access_authentication) by concatenating those values with a semicolon (`:`) and encoding the resulting string in base64. 

You will also need to provide `credential_type`, `redirect_uri` and `client_id` as the query string parameters. All those values you should have received from the Sensibill Support Team after your Client Account was set up. `client_id` is the Client Key. Please also make sure to supply the `redirect=false` parameter to avoid any actual redirects, see below.

<!--
title: "Requesting authorization code for the User Account"
-->
```curl
curl --request GET 'https://receipts-sandbox.sensibill.io/api/v1/authorizationGrant?credential_type=some_bank&response_type=code&redirect_uri=https://test-bank.example.com&client_id=vxxPt53IFll1Ua6K3ZwM1SML_lSCLpCRXftkc8bcegKo&redirect=false' \
--header 'Authorization: Basic c29tZVVzZXJuYW1lOnNvbWVQYXNzd29yZA=='
```

Where `c29tZVVzZXJuYW1lOnNvbWVQYXNzd29yZA==` is the base64 encoded User Access Id and Secret separated by a semicolon `:` (like so `base64encode(username:password)`).

<!--theme: warning-->
>You will notice that the `redirect_uri` is supplied but we also set the `redirect=false` parameter. The reason for that is that the redirecting doesn't make sense in the flow described above. Creating User Accounts, requesting authorization codes and then acquiring User Access Tokens are usually all implemented within the Integration Server on the client-side. If you think you actually may need a redirect please contact Sensibill to discuss the approach.

### Using Authorization Code to acquire User Access Tokens

With the Authorization Code at hand you can now request the User Access Token (a.k.a. User Session Token) to start using the document management API on behalf of the User Account. 

User the `POST /accessToken` endpoint and supply you Client Key and Secret in the [Basic Auth Authorization header](https://en.wikipedia.org/wiki/Basic_access_authentication) by concatenating those values with a semicolon (`:`) and encoding the resulting string in base64. In the body of the request, you will need to supply the Authorization Code acquired in the previous step as well as the grant type and redirect URI configured for your Client Account (please see the warning in the previous section about the redirect URIs).

```curl
curl --request POST 'https://receipts-sandbox.sensibill.io/api/v1/accessToken' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic dGVzdEFwaUtleTp0ZXN0QXBpU2VjcmV0' \
--data-raw '{
	"grant_type": "authorization_code",
	"redirect_uri": "https://test-bank.example.com",
	"code": "W8Qu_m13Yxx1L1xTiCwuu4zsDsc1NBfDL5YIbPa6EMnheEneh3A5o-Ycu8lmP0-BFfqP3rQbsIFUX9eaktmf"
}'
```

Where `dGVzdEFwaUtleTp0ZXN0QXBpU2VjcmV0` is the base64 encoded API Key and Secret separated by a semicolon `:` (like so `base64encode(clientKey:clientSecret)`).

## A better way

You may have noticed that the OAuth 2.0 is a bit "chatty". Indeed, creating a user account requires 2 requests, acquireing a User Access Token is another 2 requests. That may not be the best way. If you feel the same way then we highly recommend that you check out the **[JWT Auth Flow](./JWT-Authentication-Flow.md).**




