# Overview

This page contains recommendations for generating random values.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Use **only** cryptographically strong random generators to produce random values, see the [Cryptographically strong random generators](#cryptographically-strong-random-generators) section.
- Do **not** use standard pseudo-random number generators, such as those used in mathematics to generate pseudo-random numbers.

<details>
<summary>Clarification</summary>

Standard pseudo-random number generators can not withstand cryptographic attacks because such generators are based on functions that produce predictable values.
</details>

- Make sure the seed that is used to initialize a generator has enough entropy.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Periodically reinitialize seeds.

# Cryptographically strong random generators

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

{% tabs %}
{% tab title="Go" %}

Use the [crypto/rand](https://pkg.go.dev/crypto/rand) package to generate cryptographically strong random values in Go.

```go
import "crypto/rand"

func GetRandomBytes(length int) ([]byte, error) {
    r := make([]byte, length)
    if _, err := rand.Read(r); err != nil {
        return nil, err
    }
    return r, nil
}

func GetRandomString(length int) (string, error) {
    const alphabet = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
    r := make([]byte, length)
    for i := 0; i < length; i++ {
        num, err := rand.Int(rand.Reader, big.NewInt(int64(len(alphabet))))
        if err != nil {
            return "", err
        }
        r[i] = alphabet[num.Int64()]
    }
    return string(ret), nil
}
```
{% endtab %}

{% tab title="Java" %}

Use the [java.security.SecureRandom](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/security/SecureRandom.html) class to generate cryptographically strong random values.

Starting with JDK8, use `SecureRandom.getInstanceStrong()` to get a `SecureRandom` instance that implements a strong generation algorithm. For a UNIX-like OS, the default strong generation algorithm is [NativePRNGBlocking](https://docs.oracle.com/en/java/javase/15/docs/specs/security/standard-names.html), which is based on `/dev/random`. As a result, `SecureRandom.getInstanceStrong()` will return a `SecureRandom` implementation that can block a thread when the `generateSeed` or `nextBytes` methods are called. Use [NativePRNGNonBlocking](https://docs.oracle.com/en/java/javase/15/docs/specs/security/standard-names.html) to avoid possible thread blocking.

```java
// JDK8+
try {
    // UNIX-like and Windows
    // possible thread blocking for a UNIX-like OS
    SecureRandom secureRandomStrong = SecureRandom.getInstanceStrong();
    byte[] bytes = new byte[16];
    secureRandomStrong.nextBytes(bytes);
    // UNIX-like (without thread blocking)
    SecureRandom secureRandomNative = SecureRandom.getInstance("NativePRNGNonBlocking");
    byte[] bytes = new byte[16];
    secureRandomNative.nextBytes(bytes);
} catch(NoSuchAlgorithmException e) {
    throw new RuntimeException("Failed to instantiate random number generator", e);
}
```

For older versions of the JDK for UNIX-like OS, create a `SecureRandom` instance using the default constructor. However, in this case, calling the `generateSeed` method may cause thread blocking. For Windows, use [Windows-PRNG](https://docs.oracle.com/en/java/javase/15/docs/specs/security/standard-names.html).

```java
// JDK <= 7
try {
    // Unix-like
    SecureRandom secureRandom = new SecureRandom();
    byte[] bytes = new byte[16];
    secureRandom.nextBytes(bytes);
    // Windows
    SecureRandom secureRandomWindows = SecureRandom.getInstance("Windows-PRNG");
    byte[] bytes = new byte[16];
    secureRandomWindows.nextBytes(bytes);
} catch(NoSuchAlgorithmException e) {
    throw new RuntimeException("Failed to instantiate random number generator", e);
}
```

Avoid using the `SHA1PRNG` generation algorithm. If it is necessary to use `SHA1PRNG`:

1. Generate random values of length <= 20 bytes.
1. Create a new instance of `SecureRandom` for each generation.

```java
try {
    SecureRandom sha1Random = SecureRandom.getInstance("SHA1PRNG");
    byte[] bytes = new byte[20];
    sha1Random.nextBytes(bytes);
    // new instance for new generation
    sha1Random = SecureRandom.getInstance("SHA1PRNG");
    sha1Random.nextBytes(bytes); 
} catch(NoSuchAlgorithmException e) {
    throw new RuntimeException("Failed to instantiate random number generator", e);
}
```
{% endtab %}

{% tab title="Node.js" %}

Use the [crypto](https://nodejs.org/api/crypto.html) package to generate cryptographically strong random values.

```javascript
const { randomBytes } = await import('node:crypto');
const buf = randomBytes(16);
const randomHex = buf.toString('hex');
```
{% endtab %}

{% tab title="Python" %}

Use the [secrets](https://docs.python.org/3/library/secrets.html) package to generate cryptographically strong random values.

```python
import secrets

secrets.token_bytes(16)
# => b'\x1aQ\xd7\xb7\xad\t\x8aa[\xf5\xa6\x12\x89\xf5\x86\xa1'
secrets.token_hex(16)
# => '213c584a849a43da689ec65ac4f05518'
secrets.token_urlsafe(16)
# => 'KdZYoev-7aY7zpmDh_uz_w'
```
{% endtab %}
{% endtabs %}