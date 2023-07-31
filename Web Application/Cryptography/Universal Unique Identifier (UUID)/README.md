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

{% tab title="Java" %}

Use the [java.util.UUID](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/util/UUID.html) class to generate UUID values.

```java
import java.util.UUID;

public static String generateUUIDv4() {
    return UUID.randomUUID().toString();
}
```
{% endtab %}

{% tab title="Node.js" %}

Use the [uuid](https://www.npmjs.com/package/uuid) package to generate UUID values.

```javascript
import { v4 as uuidv4 } from 'uuid';
uuidv4();
```
{% endtab %}

{% tab title="Python" %}

Use the [uuid](https://docs.python.org/3/library/uuid.html) package to generate UUID values.

```python
import uuid

def generate_uuid_v4() -> str:
    return str(uuid.uuid4())
```
{% endtab %}
{% endtabs %}