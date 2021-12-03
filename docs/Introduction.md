# Introduction

The Sensibill API is structured following RESTful principles.  

There is a concept of a **Client Account** and a **User Account** in Sensibill's system. Each Client Account may have multiple User Accounts.

At the moment receipt processing and management features of the Sensibill API can only be used through a User Account. Client Accounts are only used to manage User Accounts.

![Relation between Clients, Users and Receipts](../assets/images/entity-relations.png 'Relation between Clients, Users and Receipts')

The general high-level flow is:
- A client creates a Client Account with Sensibill through the Sensibill Support Team;
- Using Client Account credentials, a client registers a User Account for each end-user on their side using the user management API (`POST /users` or `POST /jwtRegister` endpoints depending on the auth method).
- When an individual user on the client needs to do receipt processing/management through the API - a **User Access Token** must be acquired for the User Account associated with that end-user.
- That token is then submitted with each request to the Sensibill API. ([See Authentication for details.](./Authentication.md))

<!--theme: danger-->
> You cannot use a single User Account for all the end-user accounts on the client side. The requests related to the same User Account are processed in sequence and not parallelized. For instance, if you submit two images, the second one will get processed after the previous one is complete.

<!--theme: info-->
> To get your Client Account set up please contact Sensibill Support Team.

---

**Next -> [Authentication](./Authentication.md)**
