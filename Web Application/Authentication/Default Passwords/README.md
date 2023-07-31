# Overview

The page contains recommendations for working with default passwords.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Avoid setting default passwords.
- If you are setting "default" passwords:

    - Generate passwords using a cryptographically strong random generator, see the [Cryptography: Random Generators](/Web%20Application/Cryptography/Random%20Generators/README.md) page. 
    - Default passwords must follow the password policy, see the [Authentication: Password Policy](/Web%20Application/Authentication/Password%20Policy/README.md) page.
    - Default passwords must expire after a short period (for example, 7 days).
    - A user must set a new password after the first authentication with a default password.
    - Prohibit setting a default password as a long-term one.
