# Overview

This page contains recommendations for the secure use of JSON Web Tokens (JWTs).

JSON Web Token (JWT) represents a set of claims as a JSON object that is encoded in a JSON Web Signature (JWS) and/or JSON Web Encryption (JWE) structure. A JWT is represented as a sequence of URL-safe parts (JWT claims sets) separated by `.`. Each part contains a `base64url-encoded` value. For example:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

The number of parts in the JWT is dependent upon the representation of the resulting [JWS using the JWS compact serialization](#json-web-signature-jws) or [JWE using the JWE compact serialization](#json-web-encryption-jwe).

## JWT claims

The JWT claims set represents a JSON object whose members are the claims conveyed by a JWT. There are three classes of JWT claim names:

- [Registered Claim Names](#registered-claim-names)
- [Public Claim Names](#public-claim-names)
- [Private Claim Names](#private-claim-names)

### Registered claim names

Claim names in the table below are registered in the IANA "JSON Web Token Claims" registry. None of these claims are intended to be mandatory to use or implement in all cases, but rather they provide a starting point for a set of useful, interoperable claims. Applications using JWTs should define which specific claims they use and when they are required or optional.

| Claim | Definition | Type | Description |
| --- | --- | --- | --- |
| `iss` | Issuer | String or URI | The `iss` claim identifies the principal that issued the JWT |
| `sub` | Subject | String or URI | The `sub` claim identifies the principal that is the subject of the JWT |
| `aud` | Audience | <p>Array of strings (String or URI)</p><p>String or URI</p> | The `aud` claim identifies the recipients that the JWT is intended for |
| `exp` | Expiration Time | NumericDate | The `exp` claim identifies the expiration time on or after which the JWT must not be accepted for processing |
| `nbf` | Not Before | NumericDate | The `nbf` claim identifies the time before which the JWT must not be accepted for processing |
| `iat` | Issued At | NumericDate | The `iat` claim identifies the time at which the JWT was issued |
| `jti` | JWT ID | String | The `jti` claim provides a unique identifier for the JWT |

### Public claim names

Claim names can be defined at will by those using JWTs. Claim name is a value that contains a collision-resistant name.

### Private claim names

A producer and consumer of a JWT may agree to use private claim names: names that are not registered claim names or public claim names. Unlike public claim names, private claim names are subject to collision.

## JSON Object Signing and Encryption (JOSE) header

The JSON Object Signing and Encryption (JOSE) header is a JSON object containing the parameters describing the cryptographic operations and parameters employed.

For a JWT object, the members of the JSON object represented by the JOSE header describe:

- the cryptographic operations applied to the JWT,
- (optionally) additional properties of a JWT.

Depending upon whether the JWT is a [JWS](#json-web-signature-jws) or [JWE](#json-web-encryption-jwe), the corresponding rules for the JOSE header values apply.

## JSON Web Signature (JWS)

{% hint style="info" %}
JWT is a JWS using the JWS compact serialization.
{% endhint %}

JSON Web Signature (JWS) represents content secured with digital signatures or Message Authentication Codes (MACs) using JSON-based data structures. The JWS cryptographic mechanisms provide **integrity** protection for an arbitrary sequence of octets.

There are two closely related serializations for JWS which both share the same cryptographic underpinnings:

- The [JWS Compact Serialization](https://tools.ietf.org/html/rfc7515#section-7.1) is a compact, URL-safe representation intended for space-constrained environments such as HTTP Authorization headers and URI query parameters.
- The [JWS JSON Serialization](https://tools.ietf.org/html/rfc7515#section-7.2) represents JWSs as JSON objects and enables multiple signatures and/or MACs to be applied to the same content.

A JWS represents the following logical values:

![](/.gitbook/web-application/json-web-token-jwt/jwt-jws-example.png)

### JWS JOSE header

For a JWS, the members of the JSON object(s) representing the JOSE header describe:

- the digital signature or MAC applied to the JWS-protected header and payload,
- (optionally) additional properties of a JWS.

JWS defines the following registered header parameter names:

| Parameter | Definition | Type | Description |
| --- | --- | --- | --- |
| `alg` | Algorithm | String or URI | The `alg` header parameter identifies the cryptographic algorithm used to secure a JWS. You can find a list of possible values in the [JWA](https://tools.ietf.org/html/rfc7518) specification in [section 3.1](https://tools.ietf.org/html/rfc7518#section-3.1) |
| `jku` | JWK Set URL | URI | The `jku` header parameter is a URI that refers to a resource for a set of JSON-encoded public keys (as [JWK](https://tools.ietf.org/html/rfc7517) set), one of which corresponds to the key used to digitally sign the JWS |
| `jwk` | JSON Web Key | [JWK](https://tools.ietf.org/html/rfc7517) | The `jwk` header parameter is the public key that corresponds to the key used to digitally sign the JWS |
| `kid` | Key ID | String | The `kid` header parameter is a hint indicating which key was used to secure the JWS. When used with a [JWK](https://tools.ietf.org/html/rfc7517), the kid value is used to match a JWK `kid` parameter value |
| `x5u` | X.509 URL | URI | The `x5u` header parameter is a URI that refers to a resource for the [X.509 public key certificate or certificate chain](https://tools.ietf.org/html/rfc5280) in PEM-encoded form corresponding to the key used to digitally sign the JWS |
| `x5c` | X.509 Certificate Chain | [Array of certificate value strings](https://tools.ietf.org/html/rfc7515#section-4.1.6) | The `x5c` header parameter contains the [X.509 public key certificate or certificate chain](https://tools.ietf.org/html/rfc5280) corresponding to the key used to digitally sign the JWS |
| `x5t` | X.509 certificate SHA-1 thumbprint | String | The `x5t` header parameter is a base64url-encoded SHA-1 thumbprint (digest) of the DER encoding of the X.509 certificate corresponding to the key used to digitally sign the JWS |
| `x5t#S256` | X.509 Certificate SHA-256 Thumbprint | String | The `x5t#S256` header parameter is a base64url-encoded SHA-256 thumbprint (digest) of the DER encoding of the X.509 certificate corresponding to the key used to digitally sign the JWS |
| `typ` | Type | [IANA.MediaTypes](https://www.iana.org/assignments/media-types/media-types.xhtml) | The `typ` header parameter is used by JWS applications to declare the media type of this complete JWS. This is intended for use by the application when more than one kind of object could be present in an application data structure that can contain a JWS. To indicate that this object is a JWT, **recommended to use the "JWT" value** |
| `cty` | Content Type | [IANA.MediaTypes](https://www.iana.org/assignments/media-types/media-types.xhtml) | The `cty` header parameter is used by JWS applications to declare the media type [IANA.MediaTypes] of the secured content (the payload). In the normal case in which nested signing operations are not employed, the use of this header parameter is **not recommended**. In the case that nested signing is employed, this header parameter must be present; in this case, **the value must be "JWT"**, to indicate that a nested JWT is carried in this JWT |
| `crit` | Critical | Array of the header parameter names present in the JOSE header | The `crit` header parameter indicates that extensions and/or [JWA](https://tools.ietf.org/html/rfc7518) are being used that must be understood and processed |

### JWS payload

The sequence of octets to be secured (the message). The payload can contain an arbitrary sequence of octets.

### JWS signature

A digital signature or MAC over the JWS-protected header and payload. Since JWT uses JWS compact serialization, JWS unprotected headers are not used.

## JSON Web Encryption (JWE)

{% hint style="info" %}
JWT is a JWE using the JWE compact serialization.
{% endhint %}

JSON Web Encryption (JWE) represents encrypted content using JSON-based data structures. The JWE cryptographic mechanisms encrypt and provide **integrity** protection for an arbitrary sequence of octets. JWE utilizes authenticated encryption to ensure the **confidentiality** and **integrity** of plaintext and the **integrity** of the JWE-protected header.

There are two closely related serializations for JWE which both share the same cryptographic underpinnings:

- The [JWE Compact Serialization](https://tools.ietf.org/html/rfc7516#section-7.1) is a compact, URL-safe representation intended for space-constrained environments such as HTTP Authorization headers and URI query parameters.
- The [JWE JSON Serialization](https://tools.ietf.org/html/rfc7516#section-7.2) represents JWEs as JSON objects and enables the same content to be encrypted to multiple parties.

A JWE represents these logical values:

![](/.gitbook/web-application/json-web-token-jwt/jwt-jwe-example.png)

### JWE JOSE header

For a JWE, the members of the JSON object(s) representing the JOSE header describe:

- the encryption applied to the plain text,
- (optionally) additional properties of a JWE.

JWE defines the following registered header parameter names:

| Parameter | Definition | Type | Description |
| --- | --- | --- | --- |
| `alg` | Algorithm | String or URI | The `alg` header parameter identifies the cryptographic algorithm used to encrypt or determine the value of the Content Encryption Key (a symmetric key for the AEAD algorithm). You can find a list of possible values in the [JWA](https://tools.ietf.org/html/rfc7518) specification in [section 4.1](https://tools.ietf.org/html/rfc7518#section-4.1) |
| `enc` | Encryption Algorithm | String or URI | The `enc` header parameter identifies the content encryption algorithm used to perform authenticated encryption on the plaintext to produce the ciphertext and the authentication tag. This algorithm must be an AEAD algorithm with a specified key length. You can find a list of possible values in the [JWA](https://tools.ietf.org/html/rfc7518) specification in [section 5.1](https://tools.ietf.org/html/rfc7518#section-5.1) |
| `zip` | Compression Algorithm | String | The `zip` applied to the plaintext before encryption, if any |
| `jku` | JWK Set URL | URI | The `jku` header parameter is a URI that refers to a resource for a set of JSON-encoded public keys (as [JWK](https://tools.ietf.org/html/rfc7517) set), one of which the JWE was encrypted; this can be used to determine the private key needed to decrypt the JWE |
| `jwk` | JSON Web Key | [JWK](https://tools.ietf.org/html/rfc7517) | The `jwk` header parameter is the public key that corresponds to the key used to encrypt the JWE; this can be used to determine the private key needed to decrypt the JWE |
| `kid` | Key ID | String | The `kid` header parameter is a hint indicating which key was used to encrypt the JWE. When used with a [JWK](https://tools.ietf.org/html/rfc7517), the kid value is used to match a JWK `kid` parameter value |
| `x5u` | X.509 URL | URI | The `x5u` header parameter is a URI that refers to a resource for the [X.509 public key certificate or certificate chain](https://tools.ietf.org/html/rfc5280) in PEM-encoded form corresponding to the key used to encrypt the JWE |
| `x5c` | X.509 Certificate Chain | [Array of certificate value strings](https://tools.ietf.org/html/rfc7515#section-4.1.6) | The `x5c` header parameter contains the [X.509 public key certificate or certificate chain](https://tools.ietf.org/html/rfc5280) corresponding to the key used to encrypt the JWE |
| `x5t` | X.509 certificate SHA-1 thumbprint | String | The `x5t` header parameter is a base64url-encoded SHA-1 thumbprint (digest) of the DER encoding of the X.509 certificate corresponding to the key used to encrypt the JWE |
| `x5t#S256` | X.509 Certificate SHA-256 Thumbprint | String | The `x5t#S256` header parameter is a base64url-encoded SHA-256 thumbprint (digest) of the DER encoding of the X.509 certificate corresponding to the key used to encrypt the JWE |
| `typ` | Type | [IANA.MediaTypes](https://www.iana.org/assignments/media-types/media-types.xhtml) | The `typ` header parameter is used by JWE applications to declare the media type of this complete JWE. This is intended for use by the application when more than one kind of object could be present in an application data structure that can contain a JWE. To indicate that this object is a JWT, **recommended to use the "JWT" value** |
| `cty` | Content Type | [IANA.MediaTypes](https://www.iana.org/assignments/media-types/media-types.xhtml) | The `cty` header parameter is used by JWE applications to declare the media type [IANA.MediaTypes] of the secured content (the plaintext). In the normal case in which nested encryption operations are not employed, the use of this header parameter is **not recommended**. In the case that nested encryption is employed, this header parameter must be present; in this case, **the value must be "JWT"**, to indicate that a nested JWT is carried in this JWT |
| `crit` | Critical | Array of the header parameter names present in the JOSE header | The `crit` header parameter indicates that extensions and/or [JWA](https://tools.ietf.org/html/rfc7518) are being used that must be understood and processed |

### JWE Encrypted Key

{% hint style="info" %}
**Authenticated Encryption with Associated Data (AEAD)** is an algorithm that encrypts a plaintext, allowing additional authenticated data to be specified, and providing built-in integrity checking for the content of a ciphertext and additional authenticated data. AEAD algorithms accept two inputs, a plaintext and an additional authenticated data value, and produce two outputs, a ciphertext and an authentication tag value.

**Content encryption key** is a symmetric key for the AEAD algorithm used to encrypt a plaintext and produce a ciphertext and an authentication tag.
{% endhint %}

An encrypted value of a content encryption key. The JWE encrypted key value is specified as an empty sequence of octets for some algorithms.

### JWE Initialization Vector

An initialization vector, that is used as an initialization vector value when encrypting a plaintext. Some algorithms may not use an initialization vector, in which case this value is the empty octet sequence.

### JWE Ciphertext

A value resulting from authenticated encryption of plaintext with additional authenticated data.

### JWE Authentication Tag

{% hint style="info" %}
**Authentication Tag** is an output of an AEAD operation that ensures the integrity of the ciphertext and the additional authenticated data.
{% endhint %}

A value resulting from authenticated encryption of plaintext with additional authenticated data. Some algorithms may not use an authentication tag, in which case this value is the empty octet sequence.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Use well-known and up-to-date libraries to implement JWT support. You can find a list of libraries and a description of the provided functionality at [https://jwt.io/libraries](https://jwt.io/libraries).
- Do **not** trust any data that is passed in a JWT. Implement comprehensive input validation for each value that is read from a JWT, see the [Input Validation](/Web%20Application/Input%20Validation/README.md) page.
- JWT claims can be used to deliver a payload to exploit vulnerabilities in an application. Implement protection against these attacks in addition to input validation, see the [Vulnerability Mitigation](/Web%20Application/Vulnerability%20Mitigation/README.md) section.

<details>
<summary>Clarification</summary>

The `kid` (key ID) claim is used by the relying application to perform a key lookup. This value can contain a payload to exploit SQL or LDAP injection, Path Traversal or Command Injection vulnerabilities.

Similarly, blindly following the `jku` (JWK set URL) or `x5u` (X.509 URL) claim, which may contain an arbitrary URL, could result in Server-Side Request Forgery attacks.
</details>

- Define a list of allowed algorithms that can be used to secure JWT. Do **not** use block lists.
- Do **not** use weak or deprecated algorithms.
- Disable support for all algorithms in your library except those on an allow list.
- Validate the `alg` or `enc` claims to make sure a JWT is secured by an allowed algorithm.
- Use each key for exactly one algorithm. Validate it when the cryptographic operation is performed.

<details>
<summary>Clarification</summary>

If there is a way to force an application to use a key that is intended for a different algorithm, an attacker can abuse this behaviour to craft valid JWTs, see [Application Security Cheat Sheet: JSON Web Token Vulnerabilities - Using the same keys for different algorithms](https://0xn3va.gitbook.io/cheat-sheets/web-application/json-web-token-vulnerabilities#using-the-same-keys-for-different-algorithms)
</details>

- If the `exp` (expiration date) claim is used, validate that JWT has not expired. Reject expired JWTs.
- If the `iss` claim is used, validate that the cryptographic keys used for cryptographic operations in the JWT belong to the issuer. Reject the JWT if they do not.
- If the `sub` claim is used, validate that the subject (user) corresponds to a valid subject and/or `issuer:subject` pair in an application.
- If the same issuer can issue JWTs that are intended for use by more than one relying party or application:

    - Reject the JWT if the `aud` claim is not present.
    - Reject the JWT if the `aud` claim is not associated with the recipient.

- If the `jku` claim is used, validate that `jku` contains a URI from an allow list.

<details>
<summary>Clarification</summary>

The `jku` claim contains a URI that refers to a resource for a set of public keys (as a JWK set), one of which corresponds to the key used to digitally sign the JWS or one of which the JWE was encrypted. If the `jku` claim is not properly validated, an attacker can set the `jku` value to their server that hosts a public key. As a result, they will be able to sign or encrypt JWTs with their corresponding private key.
</details>


- If the `jwk` claim is used, validate that `jwk` contains a public key from an allow list.

<details>
<summary>Clarification</summary>

The `jwk` claim specifies a JSON Web Key (JWK) representing a public key. This public key is considered a trusted key for verification. An attacker could exploit this by forging valid JWS objects by removing the original signature, adding a new public key to the header, and then signing the object using the (attacker-owned) private key associated with the public key embedded in that JWS header.
</details>

- Reject the entire JWT if any of the validations fail (including all cryptographic operations).
- Implement proper error and exception handling to avoid situations where the absence of a signature causes JWT validation to be bypassed, see the [Error and Exception Handling](/Web%20Application/Error%20and%20Exception%20Handling/README.md) page.
- Comply with requirements from the [Cryptography: Cryptographic Keys Management](/Web%20Application/Cryptography/Cryptographic%20Keys%20Management/README.md) page.
- Comply with requirements from the [Cookie Security](/Web%20Application/Cookie%20Security/README.md) page when JWTs need to be stored securely in the browser.
- Log errors that occur when validating JWTs and working with them, see the [Logging and Monitoring](/Web%20Application/Logging%20and%20Monitoring/README.md) page.

# JWS-related

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Disable support of the `None` algorithm in production. Make sure the `None` algorithm is disabled by default.

<details>
<summary>Clarification</summary>

The `None` algorithm allows using JWT tokens without signature. An attacker can abuse the `None` algorithm support to craft arbitrary JWTs.

For example, the JWT below is a valid JWT that uses the `None` algorithm and the third part (signature) is empty. In this case, integrity protection is completely absent.

```
eyJhbGciOiJub25lIn0.eyJpc3MiOiJqb2UiLA0KICJleHAiOjEzMDA4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ.
```
</details>

- Do **not** pass any sensitive data in the JWT header or payload, see the [Sensitive Data Management](/Web%20Application/Sensitive%20Data%20Management/README.md) page.
- If a keyed-MAC algorithm is used comply with requirements from the [Hash-based Message Authentication Code (HMAC)](/Web%20Application/Cryptography/Hash-based%20Message%20Authentication%20Code%20(HMAC)/README.md) page.
- Make sure the JWT's signature is validated and it is not possible to use a JWT without a signature or with an invalid signature.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Disable support of the `None` algorithm in all environments.

# JWE-related

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Do **not** pass any sensitive data in the JWT header, see the [Sensitive Data Management](/Web%20Application/Sensitive%20Data%20Management/README.md) page.
- Make sure the internal signature is validated after decryption.

# References

- [RFC7519: JSON Web Token (JWT)](https://tools.ietf.org/html/rfc7519)
- [RFC7515: JSON Web Signature (JWS)](https://tools.ietf.org/html/rfc7515)
- [RFC7516: JSON Web Encryption (JWE)](https://tools.ietf.org/html/rfc7516)
- [RFC 8725: JSON Web Token Best Current Practices](https://www.rfc-editor.org/rfc/rfc8725.html)
