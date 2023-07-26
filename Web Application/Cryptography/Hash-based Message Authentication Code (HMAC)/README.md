# Overview

This page contains recommendations for using a Hash-based message authentication code (HMAC).

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Comply with requirements from the [Cryptography: Hashing](/Web%20Application/Cryptography/Hashing/README.md) page when choosing a hash algorithm.
- Comply with requirements from the [Cryptography: Cryptographic Keys Management](/Web%20Application/Cryptography/Cryptographic%20Keys%20Management/README.md) page when generating and storing a secret key.
- Use secret keys of length 16+ bytes.
- The length of a secret key does **not** exceed a hash block size.

| Hash algorithm | Block size, bytes |
| ---- | ---- |
| SHA-256 | 64 |
| SHA-512 | 128 |
| SHA3-256 | 136 |
| SHA3-512 | 72 |

- You can use HMAC to check the integrity (signature) of messages between internal systems.
- Do **not** use HMAC to integrate with a third-party system, use digital signatures.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

-  Use HMAC based on hash algorithms from the `SHA-2` family.

# HMAC implementation

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

{% tabs %}
{% tab title="Go" %}

Use the [crypto/hmac](https://pkg.go.dev/crypto/hmac) package to calculate HMAC in Go.

```go
import (
    "crypto/sha256"
    "crypto/hmac"
)

func CalculateHMAC(message, key []byte) []bytes {
    mac := hmac.New(sha256.New, key)
    mac.Write(message)
    return mac.Sum(nil)
}
```
{% endtab %}
{% endtabs %}