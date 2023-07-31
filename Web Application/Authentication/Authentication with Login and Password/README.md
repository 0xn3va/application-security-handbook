# Overview

This page contains recommendations for the implementation of the authentication scheme where login (username, email, etc.) and password are used as proof of identity.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Implement the authentication on the server side. In other words, all authentication decisions must be made on the server side.
- Do **not** send passwords in an URL (in the path, query, or fragment).
- Do **not** send passwords in HTTP headers.
- Do **not** use HTTP Basic Authentication.
- Do **not** use password hints or knowledge-based authentication (so-called "secret questions").
- Return the same error message for an incorrect password for an existing and non-existing username.
- Comply with requirements from the [Error and Exception Handling](/Web%20Application/Error%20and%20Exception%20Handling/README.md) page.
- Log all authentication decisions (successful and not successful), see the [Logging and Monitoring](/Web%20Application/Logging%20and%20Monitoring/README.md) page.

    - Successful authentication
    - Failed authentication
    - Successful password reset
    - Failed password reset
    - Successful password changing
    - Failed password changing

- Comply with requirements from the [Sensitive Data Management](/Web%20Application/Sensitive%20Data%20Management/README.md) page.
- Limit the number of attempts to sign in for a certain period, see the [Vulnerability Mitigation: Brute-force](/Web%20Application/Vulnerability%20Mitigation/Brute-force/README.md) page.
- Enforce multi-factor authentication, see the [Authentication: Multi-factor Authentication](/Web%20Application/Authentication/Multi-factor%20Authentication/README.md) page.
- Implement injection protection for login and password arguments, see:

    - [Vulnerability Mitigation: Command injection](/Web%20Application/Vulnerability%20Mitigation/Command%20Injection/README.md)
    - [Vulnerability Mitigation: SQL injection (SQLi)](/Web%20Application/Vulnerability%20Mitigation/SQL%20Injection/README.md)
    - [Vulnerability Mitigation: XML External Entity (XXE) Injection](/Web%20Application/Vulnerability%20Mitigation/XML%20External%20Entity%20(XXE)%20Injection/README.md)

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Notify a user via an available communication channel (email, push, SMS, etc.) about successful login under their account from an unknown location, browser, client, etc.

# Login management

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- The login must be unique. Make sure values have been truncated before checking.
- If emails are used, implement the email address validation, see the [Authentication: Email Address Confirmation](/Web%20Application/Authentication/Email%20Address%20Confirmation/README.md) page.
- If usernames are used, handle usernames as case-insensitive strings.
- If usernames are used, implement username validation, see the [Input Validation](/Web%20Application/Input%20Validation/README.md) page. The validations must include at least the following checks:

    - The username only contains alphanumeric characters and hyphens or underscores.
    - The username can not start with hyphens or underscores.
    - The username length is more than 1.
    - The username length is less than 64.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Assign usernames for users instead of using user-defined public data.
- If emails are used, ask for authentication from a user when changing email.

# Password management

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Do **not** set default passwords during registration or password reset. Allow a user to set their password.
- If you are setting default passwords, comply with recommendations from the [Authentication: Default Passwords](/Web%20Application/Authentication/Default%20Passwords/README.md) page.
- Validate passwords against the password policy, see the [Authentication: Password Policy](/Web%20Application/Authentication/Password%20Policy/README.md) page.
- Implement password comparisons in constant time. In other words, the password comparison should **not** depend on the provided values.
- Store passwords in a secure way, see the [Authentication: Password Storage](/Web%20Application/Authentication/Password%20Storage/README.md) page.
- Implement the password change, see the [Authentication: Password Change](/Web%20Application/Authentication/Password%20Change/README.md) page.
- Implement the password reset, see the [Authentication: Password Reset](/Web%20Application/Authentication/Password%20Reset/README.md) page.
- Implement a mechanism to force a user's password reset. You can reuse standard password reset for implementation of this mechanism.

<details>
<summary>Clarification</summary>

In case of password compromise, there should be a way to reset passwords for a bunch of users.
</details>

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Terminates all active sessions after a successful password change and reset. Do the termination across an application, federated login (if present), and any relying parties.
- Notify users if their password has been changed or restored. Add the `What should I do if it wasn't me?` section with a link to a password reset page and other relative pages (multi-factor authentication set-up, support contacts, etc.).

# References

- [OWASP Cheat Sheet Series: Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html#logging-and-monitoring)
