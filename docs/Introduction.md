# Introduction

## Client and User Accounts

The Sensibill API is structured following RESTful principles.  

There is a concept of a **Client Account** and a **User Account** in Sensibill's system. Each Client Account may have multiple User Accounts.

At the moment receipt processing and management features of the Sensibill API can only be used through a User Account. Client Accounts are only used to manage User Accounts (see **Figure 1**).

![Figure 1. Relation between Clients, Users and Receipts.](https://github.com/getsensibill/receipts-docs/raw/main/assets/images/entity-relations.png 'Figure 1. Relation between Clients, Users and Receipts.')

The general high-level flow is:
- A client creates a Client Account with Sensibill through the Sensibill Support Team;
- Using Client Account credentials, a client registers a User Account for each end-user on their side using the user management API (`POST /users` or `POST /jwtRegister` endpoints depending on the auth method).
- When an individual user on the client needs to do receipt processing/management through the API - a **User Access Token** must be acquired for the User Account associated with that end-user.
- That token is then submitted with each request to the Sensibill API. ([See Authentication for details.](./Authentication.md))

<!--theme: danger-->
> You cannot use a single User Account for all the end-user accounts on the client side. The requests related to the same User Account are processed in sequence and not parallelized. For instance, if you submit two images, the second one will get processed after the previous one is complete.

<!--theme: danger-->
> You should not be creating separate user accounts each time you need to process a receipt due to the concurrency management approach mentioned above. It would violate fair use policy and makes it harder to manage resources. If such use cases are detected, they may become a reason for escalation and even temporary suspension of accounts if not addressed.

<!--theme: info-->
> To get your Client Account set up please contact Sensibill Support Team.

## Integration Patterns

There are three main standard integration patterns for clients mainly revolving around the Sensibill SDK usage (for the Sensibill SDK information please refer to the docs [here](https://sensibill.github.io/spend-manager-docs/)):
- Full SDK Integration (see **Figure 2**) below.
- Standalone Mobile Capture Module with direct Sensibill API communication from the client's apps (see **Figure 3**) below.
- Standalone Mobile Capture Module with Sensibill API communication from the client's backend system (see **Figure 4**) below.

Please take some time to review the diagrams below, they should be quite self-explanatory. You will notice that in all cases there is something called **Integration Service** (in red) that needs to be implemented on the client's backend system. It's a thin layer of logic that takes care of creating Sensibill User Accounts for each end-user on the client-side and acquiring User Access Tokens for them.

<!--theme: info-->
> An **Integration Service** is part of the client's system responsible for performing Sensibill User Accounts management. It can be implemented as either a separate service within the client's backend system or as a module that is an integral part of the client's backend app. 

![Figure2. Full SDK Integration.](https://github.com/getsensibill/receipts-docs/raw/main/assets/images/integration-full-sdk.png 'Figure 2. Full SDK Integration.')

![Figure 3. Standalone Capture Module Integration with direct Sensibill API calls from the client-side app.](https://github.com/getsensibill/receipts-docs/raw/main/assets/images/integration-capture-and-direct-from-client.png 'Figure 3. Standalone Capture Module Integration with direct Sensibill API calls from the client-side app.')

![Figure 4. Standalone Capture Module Integration with direct Sensibill API calls from the client's backend.](https://github.com/getsensibill/receipts-docs/raw/main/assets/images/integration-capture-and-direct-from-backend.png 'Figure 4. Standalone Capture Module Integration with direct Sensibill API calls from the client\'s backend.')

### Custom Headers for Direct API Ingegrations

Sensibill needs to be able to monitor the usage of the Sensibill API functionality by the operating system. It helps our Product and Engineering Team to make more informed decisions.

Usually, it can be successfully achieved by looking at the `User-Agent` standard header.

However, for the communications that originate from the Integration Service and for the integration patterns where the Sensibill API gets accessed from the client's backend system (**Figure 4** above) the `User-Agent` header may not be present or may contain the information about the backend system issuing the request rather than the end-user's OS.

For that reason, Sensibill introduced a custom header called `sb-request-origin`. The purpose of the header is to capture the end-user's User Agent so it needs to be assigned the value of the original `User-Agent` header that came with the request originating on the end-user facing client side app.

<!--theme: info-->
> A custom  `sb-request-origin` header should be populated with the data from the standard `User-Agent` header of the request which originated on the end-user facing client app and sent with each request to the Sensibill API for all integrations where communication with the Sensibill API happens directly from the client's backend system.

#### Example:

- A user submits a receipt for processing from their iOS Safari browser app.
- The request lands on the client's backend system first (**Figure 4** above).
- That request contains a `User-Agent` header with the following value:
  ```
  User-Agent: Mozilla/5.0 (iPhone; CPU iPhone OS 14_6 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.1 Mobile/15E148 Safari/604.1
  ```
- The client's backend needs to send that image to Sensibill's API. The `User-Agent` header will be populated with the value describing the backend system and the custom `sb-request-origin header` will be added with the value from the original browser app's request. 
- The Sensibill API will end up receiving a request which contains the following headers:
  ```
  User-Agent: <somes erver version>
  sb-request-origin: Mozilla/5.0 (iPhone; CPU iPhone OS 14_6 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.1 Mobile/15E148 Safari/604.1
  ```


---

**Next -> [Authentication](./Authentication.md)**

