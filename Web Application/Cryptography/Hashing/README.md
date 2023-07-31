# Overview

This page contains recommendations for choosing a hashing algorithm.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- If you need to hash passwords see the the [Authentication: Password Storage](/Web%20Application/Authentication/Password%20Storage/README.md) page.
- For other data, use a hashing algorithm from the `SHA-2` or `SHA-3` family.
- Do **not** use vulnerable hashing algorithms from the list below. You can find examples of collision attacks at [https://github.com/corkami/collisions](https://github.com/corkami/collisions).

<details>
<summary>Vulnerable hash algorithms</summary>

- `MD4`
- `MD5`
- `SHA-0`
- `SHA-1`
- `HAVAL-128`
- `PANAMA`
- `RIPEMD`
</details>

# Hashing implementation

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

{% tabs %}
{% tab title="Go" %}

Use the implementation of hash algorithms from the [crypto](https://pkg.go.dev/crypto) package, such as [crypto/sha256](https://pkg.go.dev/crypto/sha256) or [crypto/sha512](https://pkg.go.dev/crypto/sha512). You can find the whole list at https://pkg.go.dev/crypto#Hash

```go
package main

import (
    "crypto/sha256"
    "fmt"
)

func main() {
    p := []byte("random string for hashing")
    hash := sha256.New()
    fmt.Printf("%x\n", hash.Sum(p))
}
```
{% endtab %}

{% tab title="Java" %}

Use the [java.security.MessageDigest](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/security/MessageDigest.html) class for hashing implementation. **MessageDigest is not thread-safe**. Use a new instance for every thread.

```java
import java.security.MessageDigest;
import java.nio.charset.StandardCharsets;
import java.security.NoSuchAlgorithmException;

public static String sha256Hash(String data) throws NoSuchAlgorithmException {
    MessageDigest digest = MessageDigest.getInstance("SHA-256");
    byte[] encodedHash = digest.digest(data.getBytes(StandardCharsets.UTF_8));
    return toHex(encodedHash)
}

public static String sha3256Hash(String data) throws NoSuchAlgorithmException {
    MessageDigest digest = MessageDigest.getInstance("SHA3-256");
    byte[] encodedHash = digest.digest(data.getBytes(StandardCharsets.UTF_8));
    return toHex(encodedHash)
}

private static String toHex(byte[] byteArray) {
    String hex = "";
    for (byte i : byteArray) {
        hex += String.format("%02x", i);
    }
    return hex;
}
```
{% endtab %}

{% tab title="Node.js" %}

Use the [crypto](https://nodejs.org/api/crypto.html) package for hashing implementation. The list of supported algorithms is dependent on the available algorithms supported by the version of OpenSSL on the platform. Use `openssl list -digest-algorithms` to display the available digest algorithms.

```javascript
const { createHash } = await import('node:crypto');

async function sha256_digest(data) {
    const hash = createHash('sha256');
    hash.update(data);
    return hash.digest('hex');
}
```
{% endtab %}

{% tab title="Python" %}

Use the [hashlib](https://docs.python.org/3/library/hashlib.html) package for hashing implementation. You can find the list of available algorithms [here](https://docs.python.org/3/library/hashlib.html#constructors).

```python
import hashlib

def sha256_digest(data: str) -> str:
    encoded_data = data.encode('utf-8')
    hash = hashlib.sha256(encoded_data)
    return hash.hexdigest()

def sha3_256_digest(data: str) -> str:
    encoded_data = data.encode('utf-8')
    hash = hashlib.sha3_256(encoded_data)
    return hash.hexdigest()
```
{% endtab %}
{% endtabs %}