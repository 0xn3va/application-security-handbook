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

{% tab title="Java" %}

Use the [javax.crypto.Mac](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/javax/crypto/Mac.html) class to calculate HMAC. You can find supported Mac algorithms at [Java Security Standard Algorithm Names: Mac Algorithms](https://docs.oracle.com/en/java/javase/17/docs/specs/security/standard-names.html#mac-algorithms).

```java
import javax.crypto.Mac;
import java.security.NoSuchAlgorithmException;
import java.security.InvalidKeyException;

public static byte[] calculateHMAC(String message, byte[] key) throws NoSuchAlgorithmException, InvalidKeyException {
    Mac hasher = Mac.getInstance("HmacSHA256");
	hasher.init(new SecretKeySpec(key, "HmacSHA256"));
    return hasher.doFinal(message.getBytes());
}
```
{% endtab %}

{% tab title="Node.js" %}

Use the [crypto](https://nodejs.org/api/crypto.html) package to calculate HMAC.

```javascript
const { createHmac } = await import('node:crypto');

async function calculateHMAC(message, key) {
    const hmac = createHmac('sha256', key);
    hmac.update(message);
    return hmac.digest('hex');
}
```
{% endtab %}

{% tab title="Python" %}

Use the [hmac](https://docs.python.org/3/library/hmac.html) package to calculate HMAC. Use [hmac.compare_digest](https://docs.python.org/3/library/hmac.html#hmac.compare_digest) function instead of the `==` operator to compare digests. Using [hmac.compare_digest](https://docs.python.org/3/library/hmac.html#hmac.compare_digest) reduces the vulnerability to timing attacks.

```python
import hmac

def calculate_hmac(message: str, key: bytes | bytearray):
    h = hmac.new(key, message, hashlib.sha256)
    return h.hexdigest()

def compare_hmac_digests(a: str | bytes, b: str | bytes) -> bool:
    return hmac.compare_digest(a, b)
```
{% endtab %}
{% endtabs %}