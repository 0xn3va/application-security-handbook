# Overview

This page contains recommendations for validation and confirmation email addresses.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Implement a validation of an email address. Include at least the following:

    - An email address contains two parts, separated by an `@` symbol.
    - An email address does not contain at least the following characters:

        - backticks `` ` ``
        - single quotes `'`
        - double quotes `"`
        - null bytes `\x00`
        - new line `\x0A`
        - new page `\x0D`
        - comma `,`
        - semicolon `:`

    - The domain part contains only letters, numbers, hyphens `-` and periods `.`.
    - The local part (before the `@`) should be no more than 63 characters.
    - The total length should be no more than 254 characters.

- Validate email address is correct and legitimate:

    - [Email confirmation link](#email-confirmation-link)
    - [Email confirmation code](#email-confirmation-code)

- Do **not** automatically sign in a user after email address confirmation. Once a user confirms the ownership of an email address, send them to a usual login mechanism (with multi-factor authentication, if enabled).
- Log successful and failed email address confirmation attempts, see the [Logging and Monitoring](/Web%20Application/Logging%20and%20Monitoring/README.md) page.
- Comply with requirements from the [Error and Exception Handling](/Web%20Application/Error%20and%20Exception%20Handling/README.md) page.

# Email confirmation link

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Send a confirmation URL to a user's email and pass a random token in the query string of the URL. For example:

    ```
    https://website.local/account/email/confirm?token=GiOzXAwlZ17NsW4CkVV8MQXtiQN9cWiY
    ```

- Generate a random token using a cryptographically strong random generator, see the [Cryptography: Random Generators](/Web%20Application/Cryptography/Random%20Generators/README.md) page.
- Use a random token length of 32+ bytes.
- Set a short expiration time for a random token (~ 24 hours).
- Use a random token once. Delete a random token or transfer it to a final status that prohibits reusing.

# Email confirmation code

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Send a One-Time Password (OTP) to a user's email.
- Comply with the requirements from the [Authentication: One Time Password (OTP)](/Web%20Application/Authentication/One%20Time%20Password%20(OTP)/README.md) page.

# References

- [OWASP Cheat Sheet Series: Input Validation - Email address validation](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html#email-address-validation)
