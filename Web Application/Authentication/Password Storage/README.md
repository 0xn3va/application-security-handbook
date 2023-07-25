# Overview

This page contains recommendations for storing user passwords.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Store user passwords in a hashing form in a database on the backend.
- Implement hashing on the backend side.
- Do **not** pass passwords as part of a session ID, JWT tokens, or other variables stored by the client side in the DOM, HTML5 Storage, etc.
- Use one of the hashing algorithms listed below to store passwords in a database. Use the hashing algorithm from the list, which is implemented out-of-the-box in your programming language. It is better to choose an algorithm lower in priority, but with an implementation in the language itself, than to use third-party libraries. Hashing algorithms (sorted by priority):

    - [Argon2](#argon2)
    - [PBKDF2](#pbkdf2)

- Generate a new unique salt for each hash iteration. Do **not** use the same salt for hashing passwords.
- Use salt of length 32+ bytes.
- Use cryptographically strong random number generators to generate salt, see the [Cryptography: Random Generators](/Web%20Application/Cryptography/Random%20Generators/README.md) page.
- Comply with requirements from the [Logging and Monitoring](/Web%20Application/Logging%20and%20Monitoring/README.md) page.
- Comply with requirements from the [Error and Exception Handling](/Web%20Application/Error%20and%20Exception%20Handling/README.md) page.

# Upgrading legacy hashes

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

Use the following approach to upgrade legacy hashes:

1. Use the existing password hashes as inputs for the new hashing algorithm.

    For example, if an application originally stored passwords as `sha256(password + salt)`, upgrade this to `argon2(sha256(password + salt), new_salt)`.

1. Replace these hashes with direct hashes of the users' passwords the next time a user logs in.
1. If a session can leave longer than a couple of weeks, plan for a smooth reset of active sessions so that users can authenticate again.

# Argon2

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Use the `Argon2id` version of the `Argon2` algorithm.
- Set the minimum memory size to `64 MB` and the minimum number of iterations to `1`. If using that amount of memory `64 MB` is not possible in some contexts increase the time parameter to compensate, for example, the minimum memory size `32 MB` and the minimum number of iterations `2`.
- Set the degree of parallelism to the number of available CPUs.
- Use a derived key of length 16+ bytes.
- Use a salt of length `>=` generated derived key length. Minimal salt length `32` bytes.

{% tabs %}
{% tab title="Go" %}

Use the [golang.org/x/crypto/argon2](https://pkg.go.dev/golang.org/x/crypto/argon2) package to implement `Argon2id` password hashing.

```go
import (
    "golang.org/x/crypto/argon2"
)

func Argon2Hash(password []byte, salt []byte) []byte {
    iterations := 1
    memorySize := 64 * 1024
    threads := 4
    keyLength := 32
    return argon2.IDKey(password, salt, iterations, memorySize, threads, keyLength)
}
```
{% endtab %}
{% endtabs %}

# PBKDF2

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Use HMAC as a pseudo-random function, see the [Cryptography: Hash-based Message Authentication Code (HMAC)](/Web%20Application/Cryptography/Hash-based%20Message%20Authentication%20Code%20(HMAC)/README.md) page.
- Make sure the maximum allowed password length does **not** exceed the size of the hash function block to avoid [HMAC collisions](https://en.wikipedia.org/wiki/PBKDF2#HMAC_collisions).
- Set the number of iterations using the following table:

| Hash algorithm | Minimum number of iterations |
| ---- | ---- |
| SHA-256 | 310.000 |
| SHA-512 | 120.000 |
| SHA3-256 | 120.000 |

- Use a derived key of length 16+ bytes.
- Use a salt of length >= generated derived key length. Minimal salt length `32` bytes.

{% tabs %}
{% tab title="Go" %}

Use the [golang.org/x/crypto/pbkdf2](https://pkg.go.dev/golang.org/x/crypto/pbkdf2) package to implement PBKDF2 password hashing. 

```go
import (
    "crypto/sha256"
    "golang.org/x/crypto/pbkdf2"
)

func PBKDF2Hash(password []byte, salt []byte) []byte {
    iterations := 310000
    keyLength := 32
    return pbkdf2.Key(password, salt, iterations, keyLength, sha256.New)
}
```
{% endtab %}
{% endtabs %}