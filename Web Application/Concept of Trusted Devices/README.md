# Overview

This page describes the concept of trusted devices that introduces an additional layer of authentication. The concept of trusted devices allows blocking the source of an attack without blocking legitimate users. To do so, an application issues a token that is tied to a user's account for each device from which there was a successful authentication. As a result, when authentication is blocked for a specific account during an attack, a legitimate user can still authenticate from trusted devices.

## Algorithm description

<details>
<summary>Legend</summary>

![][exit] - End of the algorithm.
</details>

The entry point for an authentication request:

1. If an incoming request contains a device token:

    1. Validate a device token, see the [Device token validation](#device-token-validation) section.
    1. If token validation fails, proceed to step 2.
    1. If a device token is in a lockout list, reject authentication attempt ![][exit]
    1. Authenticate a user, see the [User authentication](#user-authentication) section.

1. If authentication from untrusted clients is locked out for the account, reject authentication attempt ![][exit]
1. Authenticate a user, see the [User authentication](#user-authentication) section ![][exit]

### User authentication

1. Check user credentials.
1. If credentials are valid:

    1. Issue a new device token to a user's client, see the [Issue new device token](#issue-new-device-token) section.
    1. Proceed with authenticated user.

1. Otherwise:

    1. Register failed authentication attempt.
    1. Reject authentication attempt ![][exit]

### Register failed authentication attempt

1. Register a failed authentication attempt with at least the following information:

    1. Account.
    1. Time.
    1. Device token (if present).

1. If a device token is presented:

    1. Count the number of unsuccessful authentication attempts in `K` minutes for this specific device token.
    1. If the number of unsuccessful attempts in `K` minutes is more than `M`, put the device token in a lockout list for `N` minutes.

1. Otherwise:

    1. Count the number of unsuccessful authentication attempts in `K` minutes for this specific device token.
    1. If the number of unsuccessful attempts in `K` minutes is more than `M`, lock out all authentication attempts to this specific account from untrusted clients for `N` minutes.

### Issue new device token

1. Issue a device token using one of the following strategies:

    1. [Generate a random token and save it in a database](#random-token).
    1. [Use an HMAC-based token](#hmac-based-token).
    1. [Use a JWT](#json-web-token-jwt).

1. Set an expiration date for a device token:

    1. [Random token](#random-token): add an expiration date to a database.
    1. [HMAC-based token](#hmac-based-token): add an expiration date to a message for signing.
    1. [JWT](#json-web-token-jwt): add an expiration date to a payload.

1. Bind a device token with a specific user:

    1. [Random token](#random-token): connection with a user by user ID in a database.
    1. [HMAC-based token](#hmac-based-token): add a used ID to a message for signing.
    1. [JWT](#json-web-token-jwt): add a user ID to a payload.

### Device token validation

If one of the following checks fails the **entire** validation fails:

1. Validate that a device token is valid:

    1. [Random token](#random-token): there is a record in a database with a device token value.
    1. [HMAC-based token](#hmac-based-token): the signature is valid.
    1. [JWT](#json-web-token-jwt): the signature is valid.

1. Validate that a device token has not expired:

    1. [Random token](#random-token): check out a record in a database.
    1. [HMAC-based token](#hmac-based-token): check out a signed message.
    1. [JWT](#json-web-token-jwt): check out a payload.

1. Validate that a device token corresponds to an account in which the authentication is attempted:

    1. [Random token](#random-token): check out a record in a database.
    1. [HMAC-based token](#hmac-based-token): check out a signed message.
    1. [JWT](#json-web-token-jwt): check out a payload.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- If a client is a browser, store a device token in the `ultimate` cookie, see the [Cookie Security](/Web%20Application/Cookie%20Security/README.md) page.
- Set expiration date ~ 6 months.
- Bind a device token with a specific account.

# Random token

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Use a random token as a device token.
- Use a cryptographically strong generator to generate a device token, see the [Cryptography: Random Generators](/Web%20Application/Cryptography/Random%20Generators/README.md) page.
- Use a random token of length 16+ bytes.

# HMAC-based token

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Use a message signed with HMAC as a device token. The message must contain at least:

    - User ID or username.
    - Expiration date.
    - Nonce.

<details>
<summary>Example</summary>

```
BASE64(USER_ID + EXPIRATION_DATE + NONCE + HMAC(USER_ID + EXPIRATION_DATE + NONCE, SECRET_KEY))
```
</details>

- Generate a unique nonce for each token.
- Use cryptographically strong generators to generate a nonce, see the [Cryptography: Random Generators](/Web%20Application/Cryptography/Random%20Generators/README.md) page.
- Comply with the requirements from the [Cryptography: Hash-based Message Authentication Code (HMAC)](/Web%20Application/Cryptography/Hash-based%20Message%20Authentication%20Code%20(HMAC)/README.md) page.

# JSON Web Token (JWT)

- Use a JSON Web Token (JWT) as a device token. The payload must contain at least:

    - User ID or username.
    - Expiration date.
    - Nonce.

- Generate a unique nonce for each token.
- Use cryptographically strong generators to generate a nonce, see the [Cryptography: Random Generators](/Web%20Application/Cryptography/Random%20Generators/README.md) page.
- Comply with the requirements from the [JSON Web Token (JWT)](/Web%20Application/JSON%20Web%20Token%20(JWT)/README.md) page.

<details>
<summary>Example</summary>

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
{
  "userid": 31337,
  "nonce": "GiOzXAwlZ17NsW4CkVV8MQXtiQN9cWiY",
  "exp": 1516239022
}
```
</details>

# References

- [OWASP: Slow Down Online Guessing Attacks with Device Cookies](https://owasp.org/www-community/Slow_Down_Online_Guessing_Attacks_with_Device_Cookies)

[exit]: /.gitbook/web-application/concept-of-trusted-devices/exit-icon.svg