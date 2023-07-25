# Overview

This page contains recommendations for the implementation of the authentication scheme where a phone number and one-time password are used as proof of identity.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Implement the authentication on the server side. In other words, all authentication decisions must be made on the server side.
- Generate an OTP on the server side and send it to a user by their phone number via SMS or push.
- Comply with requirements from the [Authentication: One Time Password (OTP)](/Web%20Application/Authentication/One%20Time%20Password%20(OTP)/README.md) page.
- Return the same error message for an incorrect OTP for an existing and non-existing phone number.
- Require authentication from a user when changing a phone number. A user should enter an OTP for an old phone number and a new one.
- Comply with requirements from the [Error and Exception Handling](/Web%20Application/Error%20and%20Exception%20Handling/README.md) page.
- Log all authentication decisions (successful and not successful), see the [Logging and Monitoring](/Web%20Application/Logging%20and%20Monitoring/README.md) page.

    - Successful authentication
    - Failed authentication
    - Successful phone number change
    - Failed phone number change

- Limit the number of attempts to sign in for a certain period, see the [Vulnerability Mitigation: Brute-force](/Web%20Application/Vulnerability%20Mitigation/Brute-force/README.md) page.
- Limit the possibility of resending OTPs. In other words, each subsequent send of an OTP must increase the timeout when the OTP resend is prohibited. The timeout is reset by a successful login to an application. See the [Vulnerability Mitigation: Brute-force](/Web%20Application/Vulnerability%20Mitigation/Brute-force/README.md) page.
- Implement injection protection for login and password arguments, see:

    - [Vulnerability Mitigation: Command injection](/Web%20Application/Vulnerability%20Mitigation/Command%20Injection/README.md)
    - [Vulnerability Mitigation: SQL injection (SQLi)](/Web%20Application/Vulnerability%20Mitigation/SQL%20Injection/README.md)
    - [Vulnerability Mitigation: XML External Entity (XXE) Injection](/Web%20Application/Vulnerability%20Mitigation/XML%20External%20Entity%20(XXE)%20Injection/README.md)

- Comply with requirements from the [Sensitive Data Management](/Web%20Application/Sensitive%20Data%20Management/README.md) page.
- Enforce multi-factor authentication, see the [Authentication: Multi-factor Authentication](/Web%20Application/Authentication/Multi-factor%20Authentication/README.md) page.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Generate an OTP on the client side using a TOTP algorithm, see the [Authentication: One Time Password (OTP)](/Web%20Application/Authentication/One%20Time%20Password%20(OTP)/README.md) page.
- Notify a user via an available communication channel (email, push, SMS, etc.) about successful login under their account from an unknown location, browser, client, etc.
