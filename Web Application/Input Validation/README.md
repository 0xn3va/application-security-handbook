# Overview

This section contains recommendations for providing input validation security functionality.

{% hint style="info" %}
Do **not** use input validation as the primary method of preventing Cross-Site Scripting, SQL injection and other attacks which are covered by the [Vulnerability Mitigation](/Web%20Application/Vulnerability%20Mitigation/README.md) section. Nevertheless, input validation can make a significant contribution to reducing the impact of these attacks if implemented properly.
{% endhint %}

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Perform input validation for data from all potentially untrusted sources, including suppliers, partners, vendors, regulators, internal components or applications.
- Perform input validation for all user-controlled data, see [Data source for validation](#data-sources-for-validation).
- Perform input validation as early as possible in a data flow, preferably as soon as data is received from an external party before it is processed by an application.
- Implement input validation on the server side. Do not rely solely on validation on the client side.
- Implement centralized input validation functionality.

<details>
<summary>Clarification</summary>

It means that input validation must be implemented as a separate component, package or some kind of middleware. This is necessary to achieve ease of implementation of input validation mechanisms, their improvement and reuse.
</details>

- Implement the following validation scheme:

    - Normalize processed data, see [Normalization](#normalization).
    - Perform a syntactic validation of the processed data, see [Syntactic validation](#syntactic-validation).
    - Perform a semantic validation of the processed data, see [Semantic validation](#semantic-validation).

- Implement protection against mass parameter assignment attacks.

# Data sources for validation

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

Validate all user-controlled data that are passed in:

- Query parameters or URL/GET parameters.

    ```
    GET /items?sort_by=created_at HTTP/1.1
    Host: website.local
    Cookie: session_id=cf565bc0aaf6560a56c9e6d8632baa58
    ```

- HTTP headers.

    ```
    GET /v2/user HTTP/1.1
    Host: api.website.local
    PRIVATE-TOKEN: cf565bc0aaf6560a56c9e6d8632baa58
    ```

    ```
    GET /v2/user HTTP/1.1
    Host: api.website.local
    Authorization: Bearer cf565bc0aaf6560a56c9e6d8632baa58
    ```

- Cookies.

    ```
    GET / HTTP/1.1
    Host: website.local
    Cookie: session_id=cf565bc0aaf6560a56c9e6d8632baa58
    ```

- HTTP body.

    ```
    POST /v2/user HTTP/1.1
    Host: api.website.local
    Authorization: Bearer cf565bc0aaf6560a56c9e6d8632baa58
    Content-Type: application/json

    {
        "email": "username@domain.local"
    }
    ```

    ```
    POST /v2/user HTTP/1.1
    Host: api.website.local
    Authorization: Bearer cf565bc0aaf6560a56c9e6d8632baa58
    Content-Type: application/x-www-form-urlencoded

    email=username@domain.local
    ```

- Uploaded files, see the [File Upload](/Web%20Application/File%20Upload/README.md) section.

    ```
    POST /v2/user HTTP/1.1
    Host: api.website.local
    Authorization: Bearer cf565bc0aaf6560a56c9e6d8632baa58
    Content-Type: multipart/form-data;boundary="c93793a9523238b854bbf0f336"

    --c93793a9523238b854bbf0f336
    Content-Disposition: form-data; name="logo"; filename="image.png"

    ‰PNG...
    --c93793a9523238b854bbf0f336--
    ```

# Normalization

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

{% hint style="info" %}
If a free-form text is checked against a block list, canonical encoding is mandatory. Otherwise, it is optional. Remember that block list validation must only be used in exceptional cases, see [Semantic validation](#semantic-validation).
{% endhint %}

- Ensure all the processed data is encoded in an expected encoding (for instance, `UTF-8`) and no invalid characters are present.
- Use `NFKC` canonical encoding form to treat canonically equivalent symbols.
- Define a list of allowed Unicode characters for data input and reject input with characters outside the allowed character list. For example, avoid `Cf` (Format) Unicode characters, commonly used to bypass validation or sanitization.

<details>
<summary>Clarification</summary>

Unicode classifies characters within different categories. Unicode characters have multiple uses, and their category is determined based on the primary characteristic of the character. There are printable characters such as:

- `Lu` uppercase letters.
- `Mn` nonspacing marks, for example, accents and other letter decorations.
- `Nd` decimal numbers.
- `Po` other punctuation, for example, the Full Stop dot `.`.
- `Zs` space separator.
- `Sm` math symbol, for example, `<` or `=`.

and not printable characters such as:

- `CC` control.
- `Cf` format.

The last two categories (not printable characters) are the most used for attacks trying to bypass input validation, and therefore they should be avoided if not needed. For more information on categories, please see [https://www.unicode.org/reports/tr44/#General_Category_Values](https://www.unicode.org/reports/tr44/#General_Category_Values).

There are three approaches to handle not-allowed characters:

- **Recommended (rigorous and safe)**: reject the input completely if it has any character outside of allowed character lists.
- **Not recommended (lenient but safe)**: replace not-allowed characters with the Replacement Character `U+FFFD`.
- **Not recommended (unsafe)**: remove not-allowed characters from input. This approach could have unexpected behaviors and be used by attackers to bypass subsequent validation. Therefore, ***this approach should never be used***.

Example of an allowed character list:

- `L` (Letter, contains `Lu` | `Ll` | `Lt` | `Lm` | `Lo` categories)
- `N` (Number, contains `Nd` | `Nl` | `No` categories)
- `P` (Punctuation, contains `Pc` | `Pd` | `Ps` | `Pe` | `Pi` | `Pf` | `Po` categories)
- `S` (Symbol, contains `Sm` | `Sc` | `Sk` | `So` categories)
- `Z` (Separator, contains `Zs` | `Zl` | `Zp` categories)

</details>

# Syntactic validation

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Use data type validators built into the used web framework.
- Validate against [JSON Schema](https://json-schema.org/) and [XML Schema (XSD)](https://www.w3schools.com/xml/schema_intro.asp) for input in these formats.
- Validate input against expected data type, such as integer, string, date, etc.
- Validate input against expected value range for numerical parameters and dates. If the business logic does not define a value range, consider value range imposed by language or database.
- Validate input against minimum and/or maximum length for strings.

# Semantic validation

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Define an allow list and validate all data against this list. **Avoid** using block list validation.

<details>
<summary>Clarification</summary>

There are two main validation approaches:
- Allow list validation verifies that data complies with a known list of allowed values, anything else is considered as invalid data.
- Block list validation verifies that data does not match any known blocked values. If so, the data is considered invalid, anything else is considered valid data. **Note** that if a block list validation is used, input data **must be normalized** before any comparison, validation or processing. If normalization is not done properly, block list validation can be easily bypassed.

Unfortunately, block list validation may miss unknown bad values that an attacker could leverage to bypass the validation. For example, if an application handles IP addresses, an attacker can bypass a block list that contains `127.0.0.1` using rare IP formats:

```
127.1
0x7f.0x0.0x0.0x1
0x7f001
...
```
</details>

- Define an array of allowed values as a small set of string parameters (e.g. days of a week).
- Define a list of allowed characters such as `decimal digits` or `letters`.
- You can use regular expressions to define allowed values, see the [Regular Expressions](/Web%20Application/Regular%20Expressions/README.md) section.
- Implement file validation according to the [File Upload](/Web%20Application/File%20Upload/README.md) section.
- Implement email validation according to the [Email Address Validation](/Web%20Application/Authentication/Authentication%20with%20loging%20and%20password/email-address-validation.md).

# Server-side validation implementation

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

{% tabs %}
{% tab title="Go" %}

You can use the [go-playground/validator](https://github.com/go-playground/validator) package to implement input validation.

```go
package main

import (
    "fmt"
    "github.com/go-playground/validator/v10"
)

// User contains user information
type User struct {
    FirstName       string      `validate:"required"`
    LastName        string      `validate:"required"`
    Age             uint8       `validate:"gte=0,lte=130"`
    Email           string      `validate:"required,email"`
    FavouriteColor  string      `validate:"iscolor"` // alias for 'hexcolor|rgb|rgba|hsl|hsla'
    Addresses       []*Address  `validate:"required,dive,required"` // a person can have a home and cottage...
}

// Address houses a users address information
type Address struct {
    Street  string  `validate:"required"`
    City    string  `validate:"required"`
    Planet  string  `validate:"required"`
    Phone   string  `validate:"required"`
}

func validateStruct(validate *validator.Validate) {
    address := &Address{
        Street: "Eavesdown Docks",
        Planet: "Persphone",
        Phone: "none",
    }
    user := &User{
        FirstName: "Badger",
        LastName: "Smith",
        Age: 135,
        Email: "Badger.Smith@gmail.com",
        FavouriteColor: "#000-",
        Addresses: []*Address{address},
    }

    // returns nil or ValidationErrors ( []FieldError )
    err := validate.Struct(user)
    if err != nil {
        // this check is only needed when your code could produce
        // an invalid value for validation such as interface with nil value.
        if _, ok := err.(*validator.InvalidValidationError); ok {
            fmt.Println(err)
            return
        }
        for _, err := range err.(validator.ValidationErrors) {
            fmt.Println(err.Namespace())
            fmt.Println(err.Field())
            fmt.Println(err.StructNamespace())
            fmt.Println(err.StructField())
            fmt.Println(err.Tag())
            fmt.Println(err.ActualTag())
            fmt.Println(err.Kind())
            fmt.Println(err.Type())
            fmt.Println(err.Value())
            fmt.Println(err.Param())
            fmt.Println()
        }
        // from here you can create your own error messages in whatever language you wish
        return
    }
    // handle user
}

func validateVariable(validate *validator.Validate) {
    myEmail := "joeybloggs.gmail.com"
    errs := validate.Var(myEmail, "required,email")
    if errs != nil {
        // output: Key: "" Error:Field validation for "" failed on the "email" tag
        fmt.Println(errs)
        return
    }
    // email ok, move on
}

func main() {
    validate = validator.New()
    validateStruct(validate)
    validateVariable(validate)
}
```
{% endtab %}
{% endtabs %}

# Normalization implementation

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

{% tabs %}
{% tab title="Go" %}

You can use the [golang.org/x/text/unicode/norm](https://pkg.go.dev/golang.org/x/text/unicode/norm) package to implement normalization.

```go
package normalizaton

import (
    "fmt"
    "strings"
    "unicode"
    "unicode/utf8"
    "golang.org/x/text/unicode/norm"
)

// allowed_unicode_chars contains a list of allowed Unicode character categories
var allowed_unicode_chars = []*unicode.RangeTable{
    unicode.Letter, // L category (Letters)
    unicode.Digit, // N category (Digits)
    unicode.Punct, // P category (Punctuations)
    unicode.Symbol, // S category (Symbols)
    unicode.Space, // Z category (Spaces)
}

// hasOnlyAllowedCharacters verifies that the string s has only allowed unicode characters
func hasOnlyAllowedCharacters(s string) bool {
    for len(s) > 0 {
        r, size := utf8.DecodeLastRuneInString(s)
        if !unicode.IsOneOf(allowed_unicode_chars, r) {
            return false
        }
        s = s[:len(s)-size]
    }
    return true
}

func normalize(s string) (string, error) {
    // Verify the string s has valid UTF-8 encoding
    if !utf8.ValidString(s) {
        return s, fmt.Errorf("Invalid UTF-8 characters found")
    }
    // Normalize string to NFKC form
    normS := norm.NFKC.String(s)
    // Verifies the string s has only allowed characters
    if !hasOnlyAllowedCharacters(normS) {
        return s, fmt.Errorf("It contains characters from outside allowed categories list")
    }
    return normS, nil
}
```
{% endtab %}
{% endtabs %}

# References

- [OWASP Cheat Sheet Series: Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
