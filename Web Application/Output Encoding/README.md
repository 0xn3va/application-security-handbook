# Overview

This section contains recommendations for implementing output encoding.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

In many scenarios, an application displays data that was gathered from untrusted sources such as user input. This can lead to vulnerabilities than an adversary leverage to perform attacks such as Cross-Site Scripting. To mitigate such security issues untrusted output must be encoded properly depending on the context it is being used.

- Perform comprehensive input validation for each request, see the [Input Validation](/Web%20Application/Input%20Validation/README.md) section.
- Perform output encoding according to the context of any untrusted data before displaying it in a user interface (web browser, mobile app).

    For untrusted sources of data that must be encoded, please, see [Untrusted Data that must be encoded](#untrusted-data-that-must-be-encoded).

    Web browser engines parse content differently for the contexts below. Therefore, different output encoding strategies need to be used for each of them.

    - [HTML Context](#html-context)
    - [HTML Attribute Context](#html-attribute-context)
    - [JavaScript Context](#javascript-context)
    - [CSS Context](#css-context)
    - [URL Context](#url-context)

- Do **not** display untrusted data within dangerous contexts, see [Dangerous Contexts](#dangerous-contexts).

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

-  Use your framework's default output encoding or use a well-vetted output encoding library for each context.

# HTML Context

HTML Context is when untrusted data is displayed between two HTML tags such as `<div>` or `<p>`. For example:

```html
<div> $untrusted_data </div>
```

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- All untrusted data that is going to be displayed within an HTML context, must encode all non-alphanumeric characters with their HTML entity before printing. Form example:

| Input Character | Encoded Result | Notes |
| ---- | ---- | ----- |
| `"` | `&quot;` | Double quotation mark |
| `&` | `&amp;` | Ampersand character |
| `'` | `&apos;` | Single quotation mark (apostrophe) |
| `/` | `&#x2F;` | Slash character |
| `<` | `&lt;` | Less than |
| `>` | `&gt;` | Greater than |

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Use your framework's default HTML output encoding or use a well-vetted HTML output encoding library.

# HTML Attribute Context

HTML Attribute Context is when untrusted data is displayed within an HTML attribute value such as `<div>` or `<p>`. For example:

```html
<div attr="$untrusted_data">
```

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Close all HTML attributes with quotation marks such as `'` or `"`.

<details>
<summary>Clarification</summary>

If an HTML attribute is properly quoted, only a few characters need to be encoded. Otherwise, all non-alphanumeric characters need to be encoded by their respective HTML entity.

**Properly quoted**

```html
<div attr="$untrusted_data">
<div attr='$untrusted_data'>
```

**Not properly quoted**

```html
<div attr=$untrusted_data>
```
</details>

- Do **not** use not-safe HTML attributes with untrusted data even after output escaping.

<details>
<summary>List of not-safe HTML attributes</summary>

align, alink, alt, bgcolor, border, cellpadding, cellspacing, class, color, cols, colspan, coords, dir, face, height, hspace, ismap, lang, marginheight, marginwidth, multiple, nohref, noresize, noshade, nowrap, ref, rel, rev, rows, rowspan, scrolling, shape, span, summary, tabindex, title, usemap, valign, value, vlink, vspace, width
</details>

- All untrusted data that is going to be displayed within an HTML attribute context, must encode all non-alphanumeric characters with their HTML entity before printing. For example:

| Input Character | Encoded Result | Notes |
| ---- | ---- | ----- |
| `"` | `&quot;` | Double quotation mark |
| `&` | `&amp;` | Ampersand character |
| `'` | `&apos;` | Single quotation mark (apostrophe) |
| `<` | `&lt;` | This encoding is used to avoid an input sequence `</` from prematurely terminating a `</script>` block |

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

-  Use your framework's default HTML attribute output encoding or use a well-vetted HTML attribute output encoding library.

# JavaScript Context

JavaScript Context is when untrusted data is displayed within JavaScript. For example:

```html
<script>
    ...
    alert('$untrusted_data')
    ...
</script>

<div onmouseover="'$untrusted_data'"></div>
```

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Display untrusted data only within quoted locations.

<details>
<summary>Clarification</summary>

The only safe location to display untrusted data within JavaScript is at quoted locations. For example:

```html
<script>alert('$untrusted_data')</script>
<script>x='$untrusted_data'</script>
<div onmouseover="'$untrusted_data'"></div>
```
</details>

- Do **not** display untrusted data within dangerous JavaScript contexts.

    - Callback functions.
    - JavaScript event handlers: `onclick()`, `onerror()`, `onmouseover()`.
    - JavaScript functions that parse and execute JavaScript code: `eval()`, `setInterval()`, `setTimeout()`.

- All untrusted data that is going to be displayed within a JavaScript context, must encode all non-alphanumeric characters with their respective hexadecimal notation `\xHH`. For example:

| Input Character | Encoded Result | Notes |
| ---- | ---- | ----- |
| `"` | `\x22;` | Double quotation mark |
| `&` | `\x26;` | Ampersand character |
| `'` | `\x27;` | Single quotation mark (apostrophe) |
| `/` | `\x2F;` | Slash character |
| `\` | `\x5C;` | Backslash character |

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Use your framework's default JavaScript output encoding or use a well-vetted JavaScript output encoding library.

# CSS Context

CSS Context is when untrusted data is displayed within CSS. For example:

```html
<style> selector { property : "$untrusted_data"; } </style>
<span style="property : '$untrusted_data'">Content</span>
```

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Display untrusted data only on CSS property values. Other CSS contexts are unsafe.
- All untrusted data that is going to be displayed within a CSS context, must encode all non-alphanumeric characters in their respective hexadecimal notation `\xHH`. For example:

| Input Character | Encoded Result | Notes |
| ---- | ---- | ----- |
| `"` | `\x22;` | Double quotation mark |
| `&` | `\x26;` | Ampersand character |
| `'` | `\x27;` | Single quotation mark (apostrophe) |
| `/` | `\x2F;` | Slash character |
| `\` | `\x5C;` | Backslash character |

-  Do **not** use `expression()` function within a CSS property value. In addition, use [input validation](/Web%20Application/Input%20Validation/README.md) to ensure that untrusted data displayed in CSS property value does **not** contain the `expression()` function.

<details>
<summary>Example of using expression() within CSS property</summary>

```css
background-color: expression( alert(1) );
```
</details>

- Do **not** use JavaScript URI scheme `javascript:` within CSS URL context. In addition, use [input validation](/Web%20Application/Input%20Validation/README.md) to ensure that untrusted data displayed in CSS URL context does **not** contain the `javascript:` URL scheme.

<details>
<summary>Example of using javascript URL scheme within CSS URL context</summary>

```css
background-image: url("javascript:alert(1)");
```
</details>

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Use your framework's default CSS output encoding or use a well-vetted CSS output encoding library.

# URL Context

URL Context is when untrusted data is displayed within an URL. For example:

```html
<a href="https://website.local?test=$untrusted_data">link</a >
<img src="/images/$untrusted_data" />
```

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Encode all non-alphanumeric characters with their percentage-encoding hexadecimal representation `%xx`. For example:

| Input Character | Encoded Result | Notes |
| ---- | ---- | ----- |
| `"` | `%22;` | Double quotation mark |
| `&` | `%26;` | Ampersand character |
| `'` | `%27;` | Single quotation mark (apostrophe) |
| `/` | `%2F;` | Slash character |
| `\` | `%5C;` | Backslash character |

- After URL context encoding, perform [HTML Context](#html-context), [HTML Attribute Context](#html-attribute-context), [JavaScript Context](#javascript-context) or [CSS Context](#css-context), depending the context where the URL is being used.
- Do **not** display untrusted data within the `javascript:` URL scheme in URL context. In addition, use [input validation](/Web%20Application/Input%20Validation/README.md) to ensure that untrusted data displayed in URL context does **not** contain the `javascript:` URL scheme.

<details>
<summary>An example of using the dangerous javascript scheme</summary>

```html
<a href='javascript:doSomething()'>...</a>
```
</details>

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Use your framework's default URL output encoding or use a well-vetted URL output encoding library.

# Dangerous Contexts

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

The following locations are known as dangerous contexts even after performing output encoding, therefore, displaying untrusted data on them must be avoided.

- Do **not** display untrusted data within a script tag.

    ```html
    <script>Directly in a script</script>
    ```

- Do **not** display untrusted data within HTML comments.

    ```html
    <!-- Inside an HTML comment -->
    ```

- Do **not** display untrusted data directly in CSS.

    ```html
    <style>Directly in CSS</style>
    ```

- Do **not** display untrusted data to define an HTML attribute name.

    ```html
    <div ToDefineAnAttribute=test />
    ```

- Do **not** display untrusted data to define an HTML tag name.

    ```html
    <ToDefineATag href="/test" />
    ```

# Untrusted Data that must be encoded

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

Perform output encoding for all user-controlled data that is displayed within any of the above contexts and are passed in:

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

# Output encoding implementation

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

{% tabs %}
{% tab title="Go" %}

You can use the [html/template](https://pkg.go.dev/html/template) package to perform output encoding safely in Go. [html/template](https://pkg.go.dev/html/template) performs contextual escaping, so actions can appear within JavaScript, CSS, and URI contexts.

```html
<h1>{{.PageTitle}}</h1>
<a title='{{.}}'>
<a href="/{{.}}">
<a href="?q={{.}}">
<a onx='f("{{.}}")'>
<a style="align: {{.}}">
<a style="background: '{{.}}'>
<a style="background: url('{{.}}')>
<style>p.{{.}} {color:red}</style>
<script>var pair = {{.}};</script>
```

See [html/template](https://pkg.go.dev/html/template) documentation for more detailed use.
{% endtab %}
{% endtabs %}

# References

- [OWASP Cheat Sheet Series: Cross Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
