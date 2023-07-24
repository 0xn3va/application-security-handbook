# Overview

This page contains recommendations for using Universal Unique Identifier (UUID).

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Do **not** use UUID as a random value. UUID is a unique value, not random. There is no a way to guarantee randomness, [especially for versions != 4](https://en.wikipedia.org/wiki/Universally_unique_identifier#Versions).
- Do **not** use UUID as a session identifier.
- You can use a UUID as a unique value for objects, such as a bank card ID or upload file name.

# UUID generation

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

{% tabs %}
{% tab title="Go" %}

You can use the [google/uuid](https://pkg.go.dev/github.com/google/uuid) package to generate UUID values in Go.

```go
import "github.com/google/uuid"

func GenerateNewUUIDValue() string {
    return uuid.New().String()
}
```
{% endtab %}
{% endtabs %}