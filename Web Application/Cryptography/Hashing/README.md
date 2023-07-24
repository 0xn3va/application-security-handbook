# Overview

This page contains recommendations for choosing a hashing algorithm.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- If you need to hash passwords see the the [Authentication: Password storage](/Web%20Application/Authentication/Authentication%20with%20loging%20and%20password/password-storage.md) page.
- For other data, use a hashing algorithm from the `SHA-2` or `SHA-3` family.
- Do **not** use vulnerable hashing algorithms from the list below. You can find examples of collision attacks at https://github.com/corkami/collisions.

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
{% endtabs %}