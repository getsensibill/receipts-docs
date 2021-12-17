# Authentication

As described in the [Introduction](./Introduction.md), Sensibill API requires that every end-user on the client-side has a corresponding User Account in the Sensibill system. Those User Accounts are created under the Client Account that corresponds to the client in question.

To start using receipt processing and management features of the Sensibill API, you need to, first, acquire a **User Access Token** (sometimes interchangeably referred to as the **User Session Token**). That requires Client Level credentials which can either be API key and secret or a JWT token. Please refer to the corresponding sections of the docs for details.

Usually, the logic of creating User Accounts and getting User Access Tokens for them is handled by something we call **Integration Server**. That can either be a separate service within the client's backend system or just a module that is an integral part of the client's backend app. 

<!--theme: info-->
> An **Integration Server** is part of the client's system responsible for performing Sensibill User Accounts management. It can be implemented as either a separate service within the client's backend system or as a module that is an integral part of the client's backend app. 

You can see a high-level flow of Integration Server managing User Accounts on behalf of the client and retrieving User Access Tokens below.

![High Level User Management Flow](https://github.com/getsensibill/receipts-docs/raw/main/assets/images/user-management-flow.png 'High Level User Management Flow')

As mentioned above, Client Account credentials are required to request the User Access Tokens for the related User Accounts. Client Account can be configured with OAuth2 or JWT credentials depending on which auth flow is preferred. We recommend using JWT if the client's system already supports such a method or it's easy to implement. The biggest reason is that JWT flow involves fewer calls to the Sensibill API and is more straightforward. 

See **[JWT Authentication Flow](./JWT-Authentication-Flow.md)** and **[OAuth2 Authentication Flow](OAuth2-Authentication-Flow.md)** for details.