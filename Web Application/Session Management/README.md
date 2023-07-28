# Overview

This page contains recommendations for the implementation of session management.

{% hint style="info" %}
There are two approaches to session management: **Stateful** and **Stateless**.

- In the case of the **Stateful** approach, a session token is generated on the server side, saved to a database and passed to a client. The client uses this token to make requests to the server side. Therefore, the server-side stores the following bundle: `account_id:session_id`.
- In the case of the **Stateless** approach, a session token is generated on the server side (or by a third-party service), signed using a private key (or secret key) and passed to a client. The client uses this token to make requests to the server side. Therefore, the server side needs to store a public key (or secret key) to validate the signature.
{% endhint %}

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Use the `base` cookie format to store session IDs, see the [Cookie Security](/Web%20Application/Cookie%20Security/README.md) page.
- Do **not** store any sensitive data (tokens, credentials, PII, etc.) in a session ID.
- Use the session management built into the framework you are using instead of implementing a homemade one from scratch.
- Use up-to-date and well-known frameworks and libraries that implement session management.
- Review and change the default configuration of the framework or library you are using to enhance its security.
- Consider session IDs as untrusted data, as any other user input.
- Implement an idle or inactivity timeout for every session.

<details>
<summary>Clarification</summary>

An idle or inactivity timeout defines the amount of time a session will remain active in case there is no activity in the session, closing and invalidating the session upon the defined idle period since the last HTTP request received by an application for a given session ID.
</details>

- Implement a mechanism to allow users to actively close a session (logout) after they have finished using an application.
- Invalidate the session at least on the server side while closing a session.
- Use different session IDs and token names for pre- and post-authentication flows.

<details>
<summary>Clarification</summary>

Using different tokens allows an application to keep track of anonymous users and authenticated users without the risk of exposing or binding the user session between both states.
</details>

- Do **not** cache session IDs if caching application contents is allowed, see the [Transport Layer Protection](/Web%20Application/Transport%20Layer%20Protection/README.md) page.

<details>
<summary>Clarification</summary>

If caching application contents is allowed, session IDs can be cached as a part of the `Set-Cookie` header.
</details>

- Do **not** pass a session ID in an URL (path, query or fragment).
- Renew or regenerate a session ID after any privilege level change within the associated user session (`anonymous -> regular user`, `regular user -> admin user`, etc.).

<details>
<summary>Clarification</summary>

The most common scenario where the session ID regeneration is mandatory is during the authentication process, as the privilege level of the user changes from the unauthenticated (or anonymous) state to the authenticated state. Common scenarios to include:

- Password changes.
- Permission changes.
- Switching from a regular user role to an administrator role within an application.

The session ID regeneration is mandatory to prevent session fixation attacks, where an attacker sets the session ID on the victim user's web browser instead of gathering the victim's session ID, as in most of the other session-based attacks, and independently of using HTTP or HTTPS.
</details>

- Handle and store session IDs according to the [Session Management](/Web%20Application/Session%20Management/README.md) page.
- Log successful and unsuccessful events related to a session lifecycle (such as creation, regeneration, revoke) including attempts to access resources with invalid session IDs, see the [Logging and Monitoring](/Web%20Application/Logging%20and%20Monitoring/README.md) page.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Use the `ultimate` cookie format to store session IDs, see the [Cookie Security](/Web%20Application/Cookie%20Security/README.md) page.
- If a framework is used, change the default session ID name to something neutral, for example, `sessionid` or `id`.
- Implement an absolute timeout for every session regardless of session activity.

<details>
<summary>Clarification</summary>

An absolute timeout defines the maximum amount of time a session can be active, closing and invalidating the session upon the defined absolute period since the given session was initially created by an application.
</details>

- Provide users with the ability to manage active sessions (view and close active sessions).

# Stateful approach related

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Generate a session ID using a cryptographically strong generator, see the [Cryptography: Random Generators](/Web%20Application/Cryptography/Random%20Generators/README.md) page.
- Use session IDs of length 16+ bytes.
- Do **not** accept a session ID that has never been generated by an application. In case of receiving one, generate a new one for anonymous access and set it to a user.

# Stateless approach related

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Use JSON Web Tokens (JWT) to implement stateless session management, see the [JSON Web Token (JWT)](/Web%20Application/JSON%20Web%20Token%20(JWT)/README.md) page.
- Use the `exp` claim to implement a session timeout.
- Use the following algorithm to implement the logout functionality:

    - Store the `jti` claim (unique token identifier) for each issued token.
    - If a user logged out from an application, move the `jti` to a list of blocked tokens.
    - Remove a token from the block list when a token expires (check the `exp` claim to determine if a token has expired).

# References

- [OWASP Cheat Sheet Series: Session Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
