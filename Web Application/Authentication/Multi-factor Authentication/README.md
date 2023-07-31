# Overview

This page contains recommendations for the implementation of Multi-factor authentication.

Multi-factor authentication or MFA is when a user is required to present more than one type of evidence to authenticate on an application. There are four different types of evidence (or factors) that can be used:

| Factor | Examples |
| ---- | ---- |
| Something You Know | Password, PIN. |
| Something You Have | Hardware or software token, certificate, email, SMS, phone call. |
| Something You Are | Fingerprint, face recognition. |
| Location | Source IP range, geolocation. |

Multi-factor authentication requires the presentation of evidence from two or more **different** factors. In other words, providing different evidence from the same factor (for example, passwords and PIN) is **not** multi-factor authentication.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Use a one-time password from SMS as a second authentication factor.
- Provide the option for a user to use a Time-Based One-Time Password (TOTP) generated in an authenticator application as a second authentication factor.
- Comply with requirements from the [Authentication: One Time Password (OTP)](/Web%20Application/Authentication/One%20Time%20Password%20(OTP)/README.md) page.
- If an application provides multiple ways for a user to authenticate (with a third-party service via OAuth2, in an API for mobile clients, etc.) require MFA for all those ways.
- Make sure that password reset, email confirmation, and other similar processes do **not** bypass MFA.

<details>
<summary>Clarification</summary>

A vulnerable application may grant direct access to an account without providing MFA immediately after resetting a password or following a confirmation link in an email.
</details>

- Make multi-factor authentication mandatory for high-privileged users.
- End all previously created sessions after enabling MFA.

<details>
<summary>Clarification</summary>

It is necessary to end all active sessions after enabling MFA for a user to be able to strengthen the protection of their account and "kick an attacker out" of sessions in case of account compromise.
</details>

- Require authentication data (a second authentication factor) from a user to confirm disabling multi-factor authentication.
- Comply with requirements from the [Error and Exception Handling](/Web%20Application/Error%20and%20Exception%20Handling/README.md) page.
- Log authentication attempts, multi-factor enabling and disabling, see the [Logging and Monitoring](/Web%20Application/Logging%20and%20Monitoring/README.md) page.
- If the "remember me" function is implemented:

    - Use a random token to remember the use of MFA.
    - Generate a random token using a cryptographically strong random generator, see the [Cryptography: Random Generators](/Web%20Application/Cryptography/Random%20Generators/README.md) page.
    - Use different random tokens for each user and session.
    - Store a random token in the `ultimate` cookie, see the [Cookie Security](/Web%20Application/Cookie%20Security/README.md) page.

- Do **not** rely on user-controlled values from HTTP headers to determine the user's IP address if an IP address is used as a factor.

<details>
<summary>Clarification</summary>

If an application uses user-controlled values from HTTP headers an attacker can easily spoof an IP address by sending their IP address in the following headers:

```
forwarded-for
x-forwarded-for
x-forwarder-for
x-forwarder-ip
x-client-ip
...
```
</details>

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Use a Time-Based One-Time Password (TOTP) generated in an authenticator application as a second authentication factor.
- Provide the option for a user to use a hardware token as a second authentication factor.
- Make multi-factor authentication mandatory for all users.
- Notify a user when multi-factor authentication is enabled or disabled.
- If the "remember me" function is implemented:

    - Use a random token and location/browser fingerprint/device token to remember the use of MFA.
    - Implement the functionality of remote revocation of remembered devices.

- Require MFA to confirm sensitive actions (disabling security mechanisms, API token generation, etc.) or actions that require elevated privileges.

# References

- [OWASP Cheat Sheet Series: Multifactor Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)
