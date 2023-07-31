# Overview

This page contains recommendations for the security of sensitive data stored in cookies, as well as a description of the attributes and prefixes of cookies that make them secure.

# Recommended cookie format

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Use the `base` cookie format to store sensitive data (session IDs, tokens, etc.):

    ```
    Set-Cookie: SessionID=1337...;Secure;HttpOnly
    ```

    The `base` cookie has at least `Secure` and `HttpOnly` attributes.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

-  Use the `ultimate` cookie format to store sensitive data (session IDs, tokens, etc.):

    ```
    Set-Cookie: __Host-SessionID=1337...;Path=/;Secure;HttpOnly;SameSite=Strict
    ```

    The `base` cookie has at least:

    - ` __Host` prefix.
    - `SameSite` attribute is set to `Strict`.
    - `Path` attribute is set to `/`.
    - `Secure` and `HttpOnly` attributes are set.

# Secure attribute

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

Indicates that the cookie is sent to the server only when a request is made with the `https:` scheme (except on localhost). The `Secure` attribute only protects the **confidentiality** of a cookie against MiTM attackers - there is no integrity protection.

{% hint style="warning" %}
The `Secure` attribute does not prevent all access to information in cookies. Cookies with this attribute can still be read/modified either with access to the client's disk or from JavaScript if the `HttpOnly` cookie attribute is not set.
{% endhint %}

Browser-specific details implementation:

- Insecure sites `http:` can not set cookies with the `Secure` attribute (since Chrome 52 and Firefox 52).
- The `https:` scheme requirements are ignored when the `Secure` attribute is set by localhost (since Firefox 75).

# HttpOnly attribute

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

Cookies marked with the `HttpOnly` attribute are not accessible from JavaScript, for example, through the [Document.cookie](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie) property. The `HttpOnly` attribute only protects the **confidentiality** of a cookie, but `HttpOnly` cookies can be modified by [overflowing the cookie jar from JavaScript](https://www.sjoerdlangkemper.nl/2020/05/27/overwriting-httponly-cookies-from-javascript-using-cookie-jar-overflow/).

# Path attribute

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

Indicates the path that must exist in the requested URL for the browser to send the Cookie header.

The `Path` attribute limits the scope of a cookie to a specific path on a server and can therefore be used to prevent unauthorized access to it from other applications on the same host.

The forward slash `/` character is interpreted as a directory separator, and subdirectories are matched as well. For example, for `Path=/docs`:

- The request paths `/docs`, `/docs/`, `/docs/Web/`, and `/docs/Web/HTTP` will all match.
- The request paths `/`, `/docsets`, `/fr/docs` will not match.

{% hint style="warning" %}
The [RFC6265](https://datatracker.ietf.org/doc/html/rfc6265#section-5.4) standard defines the order of cookies:

```
2. The user agent SHOULD sort the cookie-list in the following order:
 * Cookies with longer paths are listed before cookies with shorter paths.
 * Among cookies that have equal-length path fields, cookies with earlier creation-times are listed before cookies with later creation-times.
```

If an application uses only the first cookie, it is possible to force the application to use a specific cookie by adding the Path attribute with longer path.
{% endhint %}

## Cookie scope vs Same-origin policy

![](/.gitbook/web-application/cookie-security/scope-sop-cookie.png)

## Isolation of two different applications on a shared host

![](/.gitbook/web-application/cookie-security/isolation-sop-cookie.png)

# Domain attribute

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

The `Domain` attribute defines the host to which the cookie will be sent.

- If the `Domain` attribute is unspecified, it defaults to the host of the current document location, **excluding subdomains**.
- If the `Domain` attribute is specified, cookies will be sent to that domain and **all its subdomains**.

Multiple host/domain values are not allowed.

{% hint style="warning" %}
IE will always submit to subdomains regardless of the presence of the Domain attribute
{% endhint %}

# Expires attribute

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

The `Expires` attribute indicates the maximum lifetime of the cookie as an HTTP-date timestamp. After exceeding this lifetime, the cookie will be removed from the browser's cookie storage. However, it does **not** invalidate a session token that was stored in that cookie.

If unspecified, the cookie becomes a session cookie. A session finishes when the client shuts down, after which the session cookie is removed.

{% hint style="warning" %}
When user privacy is a concern, it is important that any web app implementation invalidate cookie data after a certain timeout instead of relying on the browser to do it. Many browsers let users specify that cookies should never expire.

Moreover, many web browsers have a session restore feature that will save all tabs and restore them the next time the browser is used. Session cookies will also be restored, as if the browser was never closed.
{% endhint %}

# Max-Age attribute

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

The `Max-Age` attribute indicates the number of seconds until the cookie expires. A zero or negative number will expire the cookie immediately.

{% hint style="info" %}
If both `Expires` and `Max-Age` are set, `Max-Age` has precedence.
{% endhint %}

# SameSite attribute

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

The `SameSite` attribute prevents the browser from sending cookies along with cross-site requests. The `SameSite` attribute can have one of two values (case-insensitive):

- `Strict`, means that the browser sends the cookie only for same-site requests, that is, requests originating from the same site that set the cookie. If a request originates from a URL different from the current one, no cookies with the `SameSite=Strict` attribute are sent.
- `Lax`, means that the cookie is not sent on cross-site requests, such as on requests to load images or frames, but is sent when a user is navigating to the origin site from an external site using safe HTTP methods (for example, when following a link). This is the default behavior if the `SameSite` attribute is not specified. The `safe` methods: `GET`, `HEAD`, `OPTIONS` and `TRACE`.
- `None`, means that the browser sends the cookie with both cross-site and same-site requests. Cookies with `SameSite=None` must specify the `Secure` attribute (in other words, they require a secure context).

{% hint style="warning" %}
The cookies without the `SameSite` in Chrome are still treated as `None` for the first 2 minutes, after which as `Lax` (it was implemented for backward compatibility). Therefore, it is possible to abuse this behaviour to achieve the Cross-Site Request Forgery attack, see [Bypass SameSite Cookies Default to Lax and get CSRF](https://medium.com/@renwa/bypass-samesite-cookies-default-to-lax-and-get-csrf-343ba09b9f2b).
{% endhint %}

{% hint style="info" %}
Keep in mind that same-site and cross-site requests are not the same thing. The `SameSite` cookie attribute is only concerned with cross-site requests. It does not affect cross-origin requests that refer to the same site, see [The great SameSite confusion](https://jub0bs.com/posts/2021-01-29-great-samesite-confusion/) article.
{% endhint %}

See the [Browser compatibility](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#browser_compatibility) table for information about specific browser implementation, check out the following rows:

- `Defaults to Lax`
- `Secure context required`
- `URL scheme-aware ("schemeful")`

# Cookie prefix

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

Cookie prefixes allow information to be passed in a cookie name prefix to ensure that certain attributes are present in the request to set a cookie.

The following prefixes are supported:

- `__Secure-` prefix tells the browser that the `Secure` attribute is required.
- `__Host-` prefix tells the browser that the `Path=/` and `Secure` attributes are required, and at the same time that the `Domain` attribute should not be present.

See the [Browser compatibility](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#browser_compatibility) table for information about specific browser implementation, check out the `Cookie prefixes` row.

# References

- [MDN Web Docs - Set-cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
