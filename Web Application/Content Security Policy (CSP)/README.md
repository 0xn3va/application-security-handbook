# Overview

This page contains recommendations for the implementation of the Content Security Policy.

Content Security Policy (CSP) controls which resources are allowed to be loaded or executed by the browser. CSP is a defense-in-depth strategy that aims to mitigate [Cross-Site Scripting](/Web%20Application/Vulnerability%20Mitigation/Cross-Site%20Scripting%20(XSS)/README.md) attacks.

## Syntax

The `Content-Security-Policy` HTTP header follows the next syntax:

```
Content-Security-Policy: <policy-directive>; <policy-directive>
```

where `<policy-directive>` consists of `<directive> <value>` with no internal punctuation.

For example:

```
Content-Security-Policy: default-src 'self'; frame-ancestors 'self'; form-action 'self'; object-src 'none';
```

## Directives

{% hint style="info" %}
You can find all directives and their descriptions at [MDN Web Docs: Content-Security-Policy - Directives](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy#directives).
{% endhint %}

CSP directives are divided into several classes:

- [Fetch directives](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy#fetch_directives) control the locations from which certain resource types may be loaded. For example, those directives include:

    - [default-src](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/default-src) defines the default policy for other fetching directives.
    - [frame-src](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-src) specifies valid sources for nested browsing contexts loading using elements such as `<frame>` and `<iframe>`.
    - [img-src](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/img-src) specifies valid sources of images and favicons.
    - [object-src](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/object-src) specifies valid sources for the `<object>` and `<embed>` elements.
    - [script-src](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src) specifies valid sources for JavaScript and WebAssembly resources.
    - [style-src](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/style-src) specifies valid sources for stylesheets.

- [Document directives](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy#document_directives) establish the properties of a document or worker environment to which a policy applies. For example, those directives include:

    - [base-uri](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/base-uri) restricts the URLs which can be used in a document's `<base>` element.
    - [sandbox](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/sandbox) enables a sandbox for the requested resource similar to the `<iframe>` `sandbox` attribute.

- [Navigation Directives](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy#navigation_directives) instruct the browser about the locations that the document can navigate to. For example, those directives include:

    - [form-action](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/form-action) restricts the URLs which can be used as the target of form submissions from a given context.
    - [frame-ancestors](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/frame-ancestors) specifies valid parents that may embed a page using `<frame>`, `<iframe>`, `<object>`, or `<embed>`.

- [Reporting directives](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy#reporting_directives) control the reporting process of CSP violations.
- [Other directives](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy#other_directives)

## Source values

{% hint style="info" %}
You can find all source values and their descriptions at [MDN Web Docs: Content-Security-Policy - CSP source values][csp-sources].
{% endhint %}

### Keyword values

- [none][csp-sources] will not allow the loading of any resources.
- [self][csp-sources] only allow resources from the current origin.
- [strict-dynamic][csp-sources] the trust granted to a script in the page due to an accompanying `nonce` or `hash` is extended to the scripts it loads.
- [report-sample][csp-sources] requires a sample of the violating code to be included in the violation report.

### Unsafe keyword values

{% hint style="warning" %}
The use of unsafe keyword values significantly weakens the Content Security Policy.
{% endhint %}

- [unsafe-inline][csp-sources] allows using of inline resources.
- [unsafe-eval][csp-sources] allows using of dynamic code evaluation such as [eval](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/eval).
- [unsafe-hashes][csp-sources] allows enabling specific inline event handlers.


### Hosts values

- [Host][csp-sources] allows the loading of resources from a specific host, path or specific URL. For example, `website.local`, `website.local/api/`, or `https://website.local/script.js`.
- [Scheme][csp-sources] allows the loading of resources over a specific scheme. For example, `https:`, `http:`, or `data:`.

### Other values

- [nonce-*][csp-sources] is a cryptographic nonce (only used once) to allow scripts. For example, `nonce-DhcnhD3khTMePgXwdayK9BsMqXjhguVV`.
- [sha*-*][csp-sources] `sha256`, `sha384`, or `sha512`. Followed by a dash and then the `sha*` value. For example, `sha256-jzgBGA4UWFFmpOBq0JpdsySukE1FrEN5bUpoK8Z29fY=`.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Do **not** use CSP as main and single mitigation layer against Cross-Site Scripting attacks, see the [Vulnerability Mitigation: Cross-Site Scripting (XSS)](/Web%20Application/Vulnerability%20Mitigation/Cross-Site%20Scripting%20(XSS)/README.md) page.
- Define CSP as restrictive as possible. Use [CSP Evaluator](https://csp-evaluator.withgoogle.com/) to check the CSP for possible misconfigurations.

<details>
<summary>Clarification</summary>

CSP must allow loading and executing the minimum required resources. Ideally, [CSP Evaluator](https://csp-evaluator.withgoogle.com/) should not find any ways to bypass the CSP.
</details>

- Use the basic Content Security Policy, see the [Basic Content Security Policy](#basic-content-security-policy) section.
- Use the `Content-Security-Policy` HTTP header for CSP delivering, see the [Content-Security-Policy header](#content-security-policy-header) section.
- Send the `Content-Security-Policy` HTTP header in each HTTP response.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Use the strict Content Security Policy, see the [Strict Content Security Policy](#strict-content-security-policy) section.
- Use the `Content-Security-Policy-Report-Only` HTTP header to report and log policy violations, see [Content-Security-Policy-Report-Only header](#content-security-policy-report-only-header) section.

# Basic Content Security Policy

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

Use the following Content Security Policy as a starting point to define your policy:

```
Content-Security-Policy: default-src 'self'; frame-ancestors 'self'; form-action 'self'; object-src 'none';
```

This policy assumes that:

- All resources are hosted by the same domain of a document.
- There are no inline or `eval()` for scripts and style resources.
- There is no need for other websites to frame a website.
- There are no form-submissions to external websites.
- There is no need for loading plugins.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

Use more tight Content Security Policy for defining a custom policy:

```
Content-Security-Policy: default-src 'none'; script-src 'self'; connect-src 'self'; img-src 'self'; style-src 'self'; frame-ancestors 'self'; form-action 'self';
```

This policy allows images, scripts, AJAX, and CSS from the same origin and does not allow any other resources to load such as frame, media, etc.

# Strict Content Security Policy

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Use [Nonce-based CSP](#nonce-based-csp) or [Hash-based CSP](#hash-based-csp) as a start point for defining your policy:

    ```
    // nonce-based
    Content-Security-Policy: default-src 'none'; script-src 'nonce-<RANDOM>' 'strict-dynamic' https: 'unsafe-inline'; object-src 'none'; base-uri 'none';
    // hash-based
    Content-Security-Policy: default-src 'none'; script-src 'sha384-<HASH>' 'strict-dynamic' https: 'unsafe-inline'; object-src 'none'; base-uri 'none';
    ```

    Nonce- and hashed-based CSP is a more effective strategy for mitigating XSS attacks. Choosing criteria:

    - [Nonce-based CSP](#nonce-based-csp) is more easier to implement and use it on multiple projects because it always has the same structure.
    - [Hash-based CSP](#hash-based-csp) is preferable when pages are served statically or need to be cached by a user-agent or CDN. For example, single page web applications built with frameworks such as Angular, React, Vue, etc. Any changes made to a hashed inline script (including formatting) require the corresponding CSP hash header to be updated.

## Nonce-based CSP

Nonce-based CSP sets a cryptographically strong random value with each `<script>` tag that is validated by the browser during execution. Since an attacker can not guess a valid nonce they are not able to execute a malicious code because the browser will block it.

```
Content-Security-Policy: default-src 'none'; script-src 'nonce-<RANDOM>' 'strict-dynamic' https: 'unsafe-inline'; object-src 'none'; base-uri 'none';
```

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Do **not** create a middleware that replaces all script tags with `script nonce=...` because attacker-injected scripts will then get the nonces as well. You need an actual HTML templating engine to use nonces.
- Generate a nonce using a cryptographically strong generator, see the [Cryptography: Random Generators](/Web%20Application/Cryptography/Random%20Generators/README.md) page.
- Use nonces of length 16+ bytes.
- Use a new nonce for each response.

## Hash-based CSP

Hash-based CSP sets the hash of JavaScript code foe each `<script>` tag that is validated by the browser during execution. If there are any changes on that code, the hash will not match and the browser will block the execution.

```
Content-Security-Policy: default-src 'none'; script-src 'sha384-<HASH>' 'strict-dynamic' https: 'unsafe-inline'; object-src 'none'; base-uri 'none';
```

`<HASH>` must be a hash from all inline code within a legitimate inline `<script>` tag. In case there are multiple inline `<script>` tags, the syntax is as follows:

```
Content-Security-Policy: ...; script-src 'sha384-<HASH1>' 'sha384-<HASH1>' ... 'sha384-<HASHN>' 'strict-dynamic' https: 'unsafe-inline';
```

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Calculate a hash of static inline code within `<script>` tag, see the [Cryptography: Hashing](/Web%20Application/Cryptography/Hashing/README.md) page. You can use the following commands to generate a hash:

    ```bash
    cat FILENAME.js | openssl dgst -sha384 -binary | openssl base64 -A
    # or
    shasum -b -a 384 FILENAME.js | awk '{ print $1 }' | xxd -r -p | base64
    ```

- Add the [integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) attribute for static inline code within `<script>` tag.

    ```
    Content-Security-Policy: ...; script-src 'sha384-a88AwQnZB+VDvE9tvIXrMQaPlFFSUTR+nldQm1LuPXQ=' 'strictdynamic' https: 'unsafe-inline';
    ```

    ```html
    <script src="https://external-website.local/lib-0.0.1.min.js" integrity="sha384-o88AwQnZB+VDvE9tvIXrMQaPlFFSUTR+nldQm1LuPXQ="></script>
    ```

## Implementation examples

### Inline scripts

The strict CSP disables any JavaScript code placed inline in the HTML source by default. For example, the following inline code will be blocked by the strict CSP:

```html
<script>
var foo = "314"
</script>
```

Move it to an external JavaScript file and import to the target document to avoid the block by the strict CSP:

```html
<script src="app.js" nonce="${nonce}"></script>
```

### Inlince event handlers

Inline event handlers, such as `onclick`, `onmouseover`, etc. are prevented to be executed by the strict CSP. For example, the following inline event handler code will be blocked by the strict CSP:

```html
<span onclick="doThings();">A thing.</span>
```

Refactor it as shown below to avoid the block by the strict CSP:

```html
<span id="things">A thing.</span>
<script nonce="${nonce}">
    document.getElementById('things')
        .addEventListener('click', doThings);
</script>
```

### Inline JavaScript URI

Inline JavaScript URI such as `<a href="javascript: ...">` is prevented to be executed by the strict CSP. For example, the following JavaScript URI scheme code will be blocked by the strict CSP:

```html
<a href="javascript:linkClicked()">foo</a>
```

Refactor it as shown below to avoid the block by the strict CSP:

```html
<a id="foo">foo</a>
<script nonce="${nonce}">
    document.getElementById('foo')
        .addEventListener('click', linkClicked);
</script>
```

# CSP delivery

## Content-Security-Policy header

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

Passing a CSP in the [Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) HTTP header from a server is the best and secure CSP delivery that provides a full CSP defense.

```
Content-Security-Policy: default-src 'none'; script-src 'self'; connect-src 'self'; img-src 'self'; style-src 'self'; frame-ancestors 'self'; form-action 'self';
```

For more detailed examples, see the [Basic Content Security Policy](#basic-content-security-policy) and [Strict Content Security Policy](#strict-content-security-policy) sections.

## Content-Security-Policy Meta tag

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

In case the [Content-Security-Policy HTTP header](#content-security-policy-header) can not be set, you can use the `Content-Security-Policy` `<meta>` tag within HTML markup with the `http-equiv` and content attributes. Unfortunately, this meta tag is not able to use framing protections, sandboxing or CSP violation logging endpoint.

```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self'">
```

## Content-Security-Policy-Report-Only header

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

The [Content-Security-Policy-Report-Only](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only) HTTP header allows reporting CSP violations without enforce them. This means, it can be used as a step to c reate, test and tight a CSP to fit your application needs without breaking functionality. Once you have found the best CSP directives and values for your application, you can deploy it as the `Content-Security-Policy` header to enforce protection. However, it is possible combining both `Content-Security-Policy` and `Content-Security-Policy-Report-Only` headers, which is a good practice.

Violations from `Content-Security-Policy-Report-Only` are printed on the browser's console or delivered to a defined endpoint using `report-to` or `report-uri` directives. `report-to` directive requires the `Reporting-Endpoints` HTTP header to be set and a named endpoint such as `main-endpoint`.

```
Reporting-Endpoints: main-endpoint="https://reports.website.local/main"
Content-Security-Policy-Report-Only: script-src 'self'; object-src 'none'; report-to main-endpoint;
```

# References

- [OWASP Cheat Sheet Series - Content Security Policy Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
- [MDN Web Docs: Content-Security-Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)

[csp-sources]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/Sources#sources