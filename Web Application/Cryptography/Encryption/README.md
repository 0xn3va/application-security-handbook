# Overview

This page contains recommendations for choosing an encryption algorithm, key length, cryptographic parameters and materials, and implementation features.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Do **not** implement custom cryptographic algorithms.
- Use only public algorithms that have been proven to be strong, such as `AES`, `RSA` or `Curve25519`.
- The main criterion for choosing an encryption algorithm and the key length is the required level of security. That is, the longer data must remain encrypted, the stronger algorithm must be used. The strength of an algorithm is determined by the presence of effective attacks on it and the key length used. `Table 2: Comparable strengths` from the [Recommendation for Key Management](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-57pt1r5.pdf) compares security levels for approved algorithms and key lengths.
- The implementation of a cryptographic algorithm should be widely distributed and developed with the involvement of a cryptographic expert.
- Use cryptographic strong random number generators to generate all random values that are used as cryptographic parameters such as initialization vectors, nonces, keys, etc., see the [Cryptography: Random Generators](/Web%20Application/Cryptography/Random%20Generators/README.md) page.
- Use nonces, initialization vectors, and other single-use numbers only once with a given encryption key.
- Comply with the requirements from the [Cryptography: Cryptographic Keys Management](/Web%20Application/Cryptography/Cryptographic%20Keys%20Management/README.md) page.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Use [FIPS 140-2](https://csrc.nist.gov/files/pubs/fips/140-2/upd2/final/docs/fips1402annexa.pdf) or [PCI DSS](https://www.pcisecuritystandards.org/glossary/#Strong%20Cryptography) certified implementations of cryptographic algorithms.

# Symmetric

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- The minimum allowed symmetric encryption algorithm is `AES/128/GCM`.
- Use only [NIST-approved](https://csrc.nist.gov/projects/block-cipher-techniques/bcm) encryption modes such as `CCM`, `GCM`, `CTR`, or `CBC`.
- Do **not** use [ECB encryption mode](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Electronic_codebook_(ECB)).
- When using Authenticated Encryption and Authenticated Encryption with Associated Data encryption forms:

    - Use `CCM` or `GCM` encryption modes.
    - If `CCM` and `GCM` modes are not available:

        - Use block encryption in `CBC` mode and the `Encrypt-then-MAC` technique with the [Hash-based message authentication code (HMAC)](/Web%20Application/Cryptography/Hash-based%20Message%20Authentication%20Code%20(HMAC)/README.md).
        - Do **not** use `CBC-MAC` with [variable-length data](https://en.wikipedia.org/wiki/CBC-MAC#Security_with_fixed_and_variable-length_messages).

- Use nonces, initialization vectors, and other single-use numbers only once with a given encryption key.
- Use `PKCS7` padding.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- The minimum allowed symmetric encryption algorithm is `AES/256/GCM`.
- Use Authenticated Encryption and Authenticated Encryption with Associated Data encryption forms.

# Asymmetric

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- The minimum allowed asymmetric encryption algorithm is `RSA/2048/SHA256`.
- Use elliptical curve cryptography (ECC) with a secure curve that provides at least 128 bits of security strength, such as `secp256r1`.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- The minimum allowed asymmetric encryption algorithm is `RSA/3072/SHA256`.
- Use elliptical curve cryptography (ECC) with a secure curve that provides at least 256 bits of security strength, such as `secp521r1`.

# Encryption implementation

{% tabs %}
{% tab title="Go" %}

Use the [crypto](https://pkg.go.dev/crypto) package to implement cryptographic operations.

**AES/256/GCM encryption and decryption**

```go
import (
    "crypto/aes"
    "crypto/cipher"
)

func Encrypt(aes256key []byte, nonce []byte, plaintext []byte) ([]byte, error) {
    // The key argument should be either 16, 24, or 32 bytes to select AES-128, AES-192, or AES-256
    block, err := aes.NewCipher(aes256key)
    if err != nil {
        return nil, err
    }
    aesgcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, err
    }
    return aesgcm.Seal(nil, nonce, plaintext, nil), nil
}

func Decrypt(aes256key []byte, nonce []byte, ciphertext []byte) ([]byte, error) {
    // The key argument should be either 16, 24, or 32 bytes to select AES-128, AES-192, or AES-256
    block, err := aes.NewCipher(aes256key)
    if err != nil {
        return nil, err
    }
    aesgcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, err
    }
    plaintext, err := aesgcm.Open(nil, nonce, ciphertext, nil)
    if err != nil {
        return nil, err
    }
    return plaintext, nil
}
```

**RSA/2048/SHA256 encryption and decryption**

```go
import (
    "crypto/rsa"
    "crypto/sha256"
    "crypto/rand"
)

func Encrypt(rsa2048Key *rsa.PublicKey, plaintext []byte, label []byte) ([]byte, error) {
    // crypto/rand.Reader is a good source of entropy for randomising the encryption function
    rng := rand.Reader
    hash := sha256.New()
    ciphertext, err := rsa.EncryptOAEP(hash, rng, rsa2048Key, plaintext, label)
    if err != nil {
        return nil, err
    }
    return ciphertext, nil
}

func Decrypt(rsa2048Key *rsa.PrivateKey, ciphertext []byte, label []byte) ([]byte, error) {
    // crypto/rand.Reader is a good source of entropy for randomising the encryption function
    rng := rand.Reader
    hash := sha256.New()
    plaintext, err := rsa.DecryptOAEP(hash, rng, rsa2048Key, ciphertext, label)
    if err != nil {
        return nil, err
    }
    return plaintext, nil
}
```
{% endtab %}
{% endtabs %}