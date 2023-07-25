# Overview

This section contains recommendations for implementing and using functionality related to cryptographic operations.

{% hint style="warning" %}
Main rule of cryptography is do **not** invent your own cryptography. It can definitely be hacked.
{% endhint %}

# General practices

| Scenario | Algorithm (base) | Algorithm (advanced) |
| ---- | ---- | ----- |
| Key exchange | Diffie-Hellman key exchange, 2048 bit | ECDH Curve25519 |
| Message integrity | HMAC-SHA2, 256 bit | HMAC-SHA2, 512 bit |
| Message hash | SHA2, 256 bit | SHA2, 512 bit |
| Asymmetric encryption | RSA, 2048 bit, SHA-256 | ECC Curve25519 or RSA, 3072 bit, SHA-256 |
| Symmetric encryption | AES, 128 bit, GCM | AES, 256 bit, GCM |
| Key exchange | Argon2 or PBKDF2 | Argon2 or PBKDF2 |
