# Overview

This page contains recommendations for the implementation of password change functionality.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Request at least the following data during password change:

    - Old password.
    - New password.
    - Confirmation of the new password.

- Terminate all active sessions after changing a password.
- Implement the CSRF protection, see the [Vulnerability Mitigation: Cross-Site Request Forgery (CSRF)](/Web%20Application/Vulnerability%20Mitigation/Cross-Site%20Request%20Forgery%20(CSRF)/README.md) page.
- Log successful and failed password change attempts, see the [Logging and Monitoring](/Web%20Application/Logging%20and%20Monitoring/README.md) page.
- Comply with requirements from the [Error and Exception Handling](/Web%20Application/Error%20and%20Exception%20Handling/README.md) page.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Limit the number of attempts to change the password for a certain period, see the [Vulnerability Mitigation: Brute-force](/Web%20Application/Vulnerability%20Mitigation/Brute-force/README.md) page.
- Ask for a second factor when a user changes a password, if a multi-factor authentication is enabled.
