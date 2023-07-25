# Overview

This page contains recommendations for using One-Time Passwords (OTPs).

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- If you need to generate OTP on the server side and send it to a user via one of the communication channels (email, SMS, push, etc.), use a cryptographically strong random generator, see the [Cryptography: Random Generators](/Web%20Application/Cryptography/Random%20Generators/README.md) page.
- If you need to generate OTP on the client side, use time-based one-time passwords (TOTPs), see the [Time-Based One-Time Password Algorithm (TOTP)](#time-based-one-time-password-algorithm-totp) section.
- Use an OTP once. Delete an OTP or transfer it to a final status that prohibits reusing.
- Link an OTP to an individual user in a database.
- If an application generates OTPs on the server side, limit the number of generation attempts in a certain period, see the [Vulnerability Mitigation: Brute-force](/Web%20Application/Vulnerability%20Mitigation/Brute-force/README.md) page.
- Limit the number of OTP verification attempts for a certain period, see the [Vulnerability Mitigation: Brute-force](/Web%20Application/Vulnerability%20Mitigation/Brute-force/README.md) page.
- Use an OTP of length 6+ symbols.
- Set a short expiration time for an OTP (~ 5 hours).
- Make sure the generation of new OTP invalidates previous ones.
- Store OTPs in a separate database, access to which must be limited. Do **not** store OTPs next to other application data.
- Do **not** send OTPs in an URL (in the path, query, or fragment).
- Comply with requirements from the [Sensitive Data Management](/Web%20Application/Sensitive%20Data%20Management/README.md) page.
- Comply with requirements from the [Logging and Monitoring](/Web%20Application/Logging%20and%20Monitoring/README.md) page.
- Comply with requirements from the [Error and Exception Handling](/Web%20Application/Error%20and%20Exception%20Handling/README.md) page.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Store OTPs in encrypted form and decrypt only certain records when accessed.
- Set a short expiration time for an OTP (~ 1 minute).
- Include information about the confirmed action in the text templates of messages with OTPs. For example, if a user uses OTP to cancel a subscription, the message can be like this:

    ```
    Use this code: 133713 to confirm cancellation of subscription #13. For more information https://website.local/yJri7a
    ```

# Time-Based One-Time Password Algorithm (TOTP)

Time-Based One-Time Password Algorithm (TOTP) is described in [RFC6238](https://datatracker.ietf.org/doc/html/rfc6238).

TOTP is an algorithm that generates OTPs based on a shared secret and the current time. To generate OTPs, a user needs an authenticator application (also known as a `prover`). In other words, a user and a backend generate an OTP separately, but thanks to the shared secret and the current time, a backend can reproduce the same OTP in a certain period.

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Generate a shared secret using a cryptographically strong random generator, see the [Cryptography: Random Generators](/Web%20Application/Cryptography/Random%20Generators/README.md) page.
- Use a unique shared secret for each authenticator application (prover). For example, if TOTP is used to generate OTPs for a second-factor authentication, generate a unique shared secret for each TOTP registration.
- Use a shared secret of length 16+ bytes.
- Set a short expiration time for an OTP (~ 1 minute).

<details>
<summary>Clarification</summary>

Since new OTPs are generated in steps of `n` seconds (generally `n` is `30` seconds), multiple OTPs are valid at the same time. Each OTP in the allowable window is a valid OTP. If OTPs are valid for an hour, many OTPs are valid at the same time within this window. Therefore, it is much easier to guess a valid OTP, because there is more than one valid OTP in the possible set of OTPs.

For example, if OTPs are generated in steps of `30` seconds and a common validity range here is about `5` minutes in either direction from the time as set on a server, the allowable window is:

```
[current_time - 5 mins, current_time + 5 mins]
```

In such case, the number of valid codes at the same time is:

```
10 mins * 60 sec / 30 sec = 20 codes
```
</details>

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- The OTP generation step and OTP expiration time must be equal (~ 1 minute). In other words, the generation of a new OTP must invalidate previous ones.
