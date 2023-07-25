# Overview

This page contains recommendations for the implementation of password reset functionality.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Implement a password reset in one of the following ways:

    - [Password reset based on a URL token](#password-reset-based-on-a-url-token)
    - [Password reset based on a one-time password](#password-reset-based-on-a-one-time-password)

- During password reset do **not** indicate if a user with entered data exists or not. Ask for a username/email address/phone number/etc. and inform the a that if there is a user with the data entered, reset recommendations will be sent to them via the appropriate communication channel.
- Only block an old password after a user has successfully passed the password reset step.
- Do **not** automatically sign a user in. Once a user sets their new password, log out a user and send them to a usual login mechanism (with multi-factor authentication, if enabled).
- Terminate all active sessions after the password reset. You can explicitly ask a user if they want to invalidate all of their existing sessions.
- During password reset, request at least the following data:

    - New password.
    - Confirmation of the new password.

- Link a random token (or one-time password) to an individual user in a database.
- Do **not** reveal a current password in any way during the password reset.
- Do **not** reveal a random token (or one-time password) in any way during the password reset.
- Log successful and failed password reset attempts, see the [Logging and Monitoring](/Web%20Application/Logging%20and%20Monitoring/README.md) page.
- Comply with requirements from the [Error and Exception Handling](/Web%20Application/Error%20and%20Exception%20Handling/README.md) page.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Implement a password reset in the following way:

    - [Password reset based on a time-based one-time password](#password-reset-based-on-a-time-based-one-time-password)

# Password reset based on a URL token

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Send a password reset URL to a user and pass a random token in the query string of the URL. For example:

    ```
    https://website.com/account/password/reset?token=GiOzXAwlZ17NsW4CkVV8MQXtiQN9cWiY
    ```

- Generate a random token using a cryptographically strong random generator, see the [Cryptography: Random Generators](/Web%20Application/Cryptography/Random%20Generators/README.md) page.
- Use a random token of length 32+ bytes.
- Set a short expiration time for a random token (~ 24 hours).
- Use a random token once. Delete a random token or transfer it to a final status that prohibits reusing.
- Do **not** rely on the `Host` HTTP header while creating the reset URLs to avoid the [Host Header Injection](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/07-Input_Validation_Testing/17-Testing_for_Host_Header_Injection) attack. Either hardcode the URL or validate against a list of trusted domains using an allow list.
- Add the [Referrer-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy) HTTP header with the `noreferrer` value to the reset password page in order to avoid [referrer leakage](https://portswigger.net/kb/issues/00500400_cross-domain-referer-leakage).

# Password reset based on a one-time password

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Send a one-time password to a user for confirmation of the password reset.
- Comply with the requirements from the [Authentication: One Time Password (OTP)](/Web%20Application/Authentication/One%20Time%20Password%20(OTP)/README.md) page.

# Password reset based on a time-based one-time password

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Use a time-based one-time password generated on the user side for confirmation of the password reset.
- Comply with the requirements from the [Authentication: One Time Password (OTP)](/Web%20Application/Authentication/One%20Time%20Password%20(OTP)/README.md) page.

# References

- [OWASP Cheat Sheet Series: Forgot Password Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Forgot_Password_Cheat_Sheet.html)
