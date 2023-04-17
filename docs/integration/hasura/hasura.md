---
title: "hasura"
metaTitle: "Hasura | Hasura Authentication Tutorial"
metaDescription: "Learn how to integrate Casdoor with Hasura to secure your applications using JWT"
---

## What is Casdoor?

Casdoor is a UI-first Identity Access Management (IAM) / Single-Sign-On (SSO) platform based on OAuth 2.0, OIDC, SAML and CAS.

Casdoor serves both the web UI and the login requests from the application users.

Casdoor comes with features such as:
* Front-end and back-end separate architecture, developed by Golang, Casdoor supports high concurrency, provides web-based managing UI and supports multiple languages(Chinese, English).
* Casdoor supports third-party applications login, such as GitHub, Google, QQ, WeChat, etc., and supports the extension of third-party login with plugins.
* With Casbin based authorization management, Casdoor supports ACL, RBAC, ABAC, RESTful accessing control models.
* Phone verification code, email verification code and password retrieval functions.
* Accessing logs auditing and recording.
* Alibaba Cloud, Tencent Cloud, Qiniu Cloud image CDN cloud storage.
* Customizable registration, login, and password retrieval pages.
* Casdoor supports integration with existing systems by db sync, so users can transition to Casdoor smoothly.
* Casdoor supports mainstream databases: MySQL, PostgreSQL, SQL Server, etc., and supports the extension of new databases with plugins.

In this section, you will learn how to integrate Casdoor with Hasura.

## Deploy Casdoor

Firstly, the Casdoor should be deployed.

You can refer to the Casdoor official documentation for the [Server Installation](https://casdoor.org/docs/basic/server-installation).

After a successful deployment, you need to ensure:

- The Casdoor server is successfully running on **http://localhost:8000**.
- Open your favorite browser and visit **http://localhost:7001**, you will see the login page of Casdoor.
- Input `admin` and `123` to test login functionality is working fine.

Then you can quickly implement a casdoor-based login page in your own app with the following steps.

## Configure Casdoor application

1. Create or use an existing Casdoor application.
2. Add a redirect url: `http://CASDOOR_HOSTNAME/login`
   ![Casdoor Application Setting](https://github.com/RanTao123/image/blob/main/Casdoor%20Application%20Setting.png?raw=true)
3. Copy the client ID, we will need it in the following steps.

## Add user in Casdoor

Now you have the application, but not a user. That means you need to create a user and assign the role.

Go to the “Users” page and click on “Add user” in the top right corner. That opens a new page where you can add the new user.

![Pic showing the users page](https://github.com/RanTao123/image/blob/main/user.png?raw=true)

Save the user after adding a username and adding the organisation Hasura(other details are optional).

Now you need to set up a password for your user, which you can do by clicking manage your password.

Choose a password for your user and confirm it.

## Build Hasura App

Start the Hasura by docker or Hasura Cloud.

Now create a `users` table with the following columns:
* `id` of type Text (Primary Key)
* `username` of type Text

See the image below for reference.

![Picture showing how to create a table in Hasura](https://graphql-engine-cdn.hasura.io/learn-hasura/assets/graphql-hasura-authentication/keycloak/hasura-create-table.png)

The next step is to create a `user` role for the app. Users should be able to see only their records, but not the other people’s records.

Configure the `user` role as shown in the image below. For more information, read about [configuring permission rules in Hasura](https://hasura.io/docs/latest/graphql/core/auth/authorization/permission-rules/).

![Picture showing how to set permissions in Hasura](https://graphql-engine-cdn.hasura.io/learn-hasura/assets/graphql-hasura-authentication/keycloak/hasura-set-permissions.png)

This way, users cannot read other people’s records. They can only access theirs.

For testing purposes, add a dummy user. This is to ensure that when you use the JWT token, you only see your user’s details and not other users’ details.

![Picture showing how to add a table record in Hasura](https://graphql-engine-cdn.hasura.io/learn-hasura/assets/graphql-hasura-authentication/keycloak/hasura-dummy-user.png)

Now you need to set the `JWT_SECRET` in Hasura.

## Configure Hasura with Casdoor

In this step, you need to add the **HASURA_GRAPHQL_JWT_SECRET** to Hasura.

To do so, go to the Hasura docker-compose.yaml and then add the new `HASURA_GRAPHQL_JWT_SECRET` as below.

The `HASURA_GRAPHQL_JWT_SECRET` should be in the following format:

```
HASURA_GRAPHQL_JWT_SECRET: '{"claims_map": {
      "x-hasura-allowed-roles": ["user","editor"],
      "x-hasura-default-role": "user",
      "x-hasura-user-id": "userID"
    },"jwk_url":"https://door.casdoor.com/.well-known/jwks"}'
```

Save the change, and reload the docker.

![Add Clerk JWT URL to Hasura](https://github.com/RanTao123/image/blob/main/MD$GWN%5BBET2O538TG~LNZIM.png?raw=true)

## Retrieve JWT Token

Since there is no client implementation, you can get your access token by making a request by below URL:
```
http://localhost:8000/login/oauth/authorize?client_id=<client ID>>&response_type=code&redirect_uri=http%3A%2F%2Flocalhost%3A8080%2Flogin&scope=read&state=app-built-in<public certificate>>
```

Change the client ID to the ID you copied before, and input the public certificate of Cadoor which you can find in Casdoor/Certs.

Then input the username and password you create for Hasura before.

Click Sign in.

![Retrieve JWT Token](https://github.com/RanTao123/image/blob/main/login.png?raw=true)

Go back to Casdoor/Token page

![Token Page](https://github.com/RanTao123/image/blob/main/asd.png?raw=true)

Find the Username you input before then click edit

Copy the Access Token.

![Access Token](https://github.com/RanTao123/image/blob/main/access.png?raw=true)

Now you can use the access token to make the authenticated request. Hasura returned the appropriate user rather than returning all the users from the database.

![Picture showing the access token from Keycloak being used in Hasura](https://github.com/RanTao123/image/blob/main/hasura.png?raw=truehttps://github.com/RanTao123/image/blob/main/hasura.png?raw=true)