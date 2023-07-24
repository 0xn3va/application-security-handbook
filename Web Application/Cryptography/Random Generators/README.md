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
{% endtabs %}