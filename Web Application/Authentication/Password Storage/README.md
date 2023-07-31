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
- Use salts of length 32+ bytes.
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
- Use deriveds key of length 16+ bytes.
- Use salts of length `>=` generated derived key length. Minimal salt length `32` bytes.

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
    threads := 1
    keyLength := 32
    return argon2.IDKey(password, salt, iterations, memorySize, threads, keyLength)
}
```
{% endtab %}

{% tab title="Java" %}

Use the [org.bouncycastle.crypto.generators.Argon2BytesGenerator](https://javadoc.io/doc/org.bouncycastle/bcprov-jdk15on/1.68/org/bouncycastle/crypto/generators/Argon2BytesGenerator.html) class from `bouncycastle` to implement `Argon2id` password hashing.

```java
import java.nio.charset.StandardCharsets;
import org.bouncycastle.crypto.generators.Argon2BytesGenerator;
import org.bouncycastle.crypto.params.Argon2Parameters;

public static String argon2idHash(String password, String salt) {
    int iterations = 1;
    int memLimit = 64 * 1024;
    int hashLength = 32;
    int parallelism = 1;
    byte[] hash = new byte[hashLength];
    Argon2Parameters params = new Argon2Parameters
        .Builder(Argon2Parameters.ARGON2_id)
        .withVersion(Argon2Parameters.ARGON2_VERSION_13)
        .withIterations(iterations)
        .withMemoryAsKB(memLimit)
        .withParallelism(parallelism)
        .withSalt(salt.getBytes(StandardCharsets.UTF_8))
        .build();
    Argon2BytesGenerator generator = new Argon2BytesGenerator();
    generator.init(params);
    generator.generateBytes(password.toCharArray(), hash);
    return toHex(hash)
}

private static String toHex(byte[] byteArray) {
    String hex = "";
    for (byte i : byteArray) {
        hex += String.format("%02x", i);
    }
    return hex;
}
```

If you are using Spring, use the [org.springframework.security.crypto.argon2.Argon2PasswordEncoder](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/crypto/argon2/Argon2PasswordEncoder.html) class from the Spring Security Crypto library that based on `bouncycastle`:

```java
import org.springframework.security.crypto.argon2.Argon2PasswordEncoder;

public static String argon2idHash(String password) {
    Argon2PasswordEncoder arg2SpringSecurity = createArgon2PasswordEncoder();
    return arg2SpringSecurity.encode(password);
}

public static boolean argon2idMatches(String password, String passwordHash) {
    Argon2PasswordEncoder arg2SpringSecurity = createArgon2PasswordEncoder();
    return arg2SpringSecurity.matches(password, passwordHash)
}

private static Argon2PasswordEncoder createArgon2PasswordEncoder() {
    int iterations = 1;
    int memLimit = 64 * 1024;
    int hashLength = 32;
    int saltLength = 32;
    int parallelism = 1;
    return new Argon2PasswordEncoder(saltLength, hashLength, parallelism, memLimit, iterations);
}
```
{% endtab %}

{% tab title="Node.js" %}

Use the [argon2](https://www.npmjs.com/package/argon2) package to implement `Argon2id` password hashing.

```javascript
const argon2 = require('argon2');

async function argon2_hash(password) {
    return argon2.hash(password);
}

async function argon2_hash_verify(password, passwordHash) {
    return argon2.verify(passwordHash, password);
}
```
{% endtab %}

{% tab title="Python" %}

Use the [argon2-cffi](https://github.com/hynek/argon2-cffi) package to implement `Argon2id` password hashing.

```python
from typing import Literal
from argon2 import PasswordHasher

def argon2_hash(password: str | bytes) -> str:
    ph = PasswordHasher()
    return ph.hash(password)

# Note: argon2_hash_verify raises argon2.exceptions.VerificationError if verification failed
def argon2_hash_verify(password: str | bytes, passwordHash: str | bytes) -> Literal[True]:
    ph = PasswordHasher()
    return ph.verify(passwordHash, password)
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

- Use derived keys of length 16+ bytes.
- Use salts of length >= generated derived key length. Minimal salt length `32` bytes.

{% tabs %}
{% tab title="Go" %}

Use the [golang.org/x/crypto/pbkdf2](https://pkg.go.dev/golang.org/x/crypto/pbkdf2) package to implement `PBKDF2` password hashing.

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

{% tab title="Java" %}

Use the [javax.crypto.SecretKeyFactory](https://docs.oracle.com/javase/8/docs/api/javax/crypto/SecretKeyFactory.html) class to implement `PBKDF2` password hashing.

```java
import javax.crypto.spec.PBEKeySpec;
import javax.crypto.SecretKeyFactory;
import java.nio.charset.StandardCharsets;
import java.security.NoSuchAlgorithmException;
import java.security.spec.InvalidKeySpecException;

public static String pbkdf2Hash(String password, String salt) throws NoSuchAlgorithmException, InvalidKeySpecException {
    int iterations = 310000;
    int keyLength = 32 * 8;
    PBEKeySpec keySpec = new PBEKeySpec(password.toCharArray(), salt.getBytes(StandardCharsets.UTF_8), iterations, keyLength);
    SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("PBKDF2WithHmacSHA256");
    byte[] keyBytes = keyFactory.generateSecret(keySpec).getEncoded();
    return toHex(keyBytes);
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

Use the [crypto](https://nodejs.org/api/crypto.html) package to implement `PBKDF2` password hashing.

```javascript
const {
    pbkdf2,
} = await import('node:crypto');

async function pbkdf2_hash(password, salt) {
    return pbkdf2
        .pbkdf2Sync(password, salt, 310000, 32, 'sha256')
        .toString('hex');
};
```
{% endtab %}

{% tab title="Python" %}

Use the [hashlib](https://docs.python.org/3/library/hashlib.html) package to implement `PBKDF2` password hashing.

```python
from hashlib import pbkdf2_hmac

def pbkdf2_hash(password: str, salt: str) -> str:
    dk = pbkdf2_hmac('sha256', password.encode('utf-8'), salt.encode('utf-8'), 310_000)
    return dk.hex()
```
{% endtab %}
{% endtabs %}