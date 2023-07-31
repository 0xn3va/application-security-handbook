# Overview

This section contains recommendations for implementing protections on the transport layer.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Encrypt all communications between an application and clients from the Internet, such as client-side applications or third-party services.
- Use TLS to encrypt communications, see the [TLS configuration](#tls-configuration) section.
- If you need to listen on unencrypted HTTP connections on port 80, immediately redirect those connections with a permanent redirect (HTTP 301 status code) to port 443 to establish an HTTPS connection.
- Do **not** load mixed content. In other words, do **not** include in pages accessible via TLS any resources, such as JavaScript or CSS, that are loaded over unencrypted HTTP.
- Use at least the `base` cookie format to instruct the browser to only send cookies over encrypted HTTPS connections, see the [Cookie Security](/Web%20Application/Cookie%20Security/README.md) page.
- Prevent caching of sensitive data, see the [Content caching](#content-caching) section.
- Add secure HTTP headers that help to protect from downgrade attacks, cookie hijacking and other attacks, see the [Secure HTTP headers](#secure-http-headers) section.
- Implement work with certificates in accordance with the [Certificates](#certificates) section.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Encryption of all communications including between application components from different subnets.

# TLS configuration


{% hint style="info" %}
You can use the following tools to check your TLS configuration:

- Online: https://www.ssllabs.com/ssltest or https://observatory.mozilla.org/
- Offline: https://github.com/nabla-c0d3/sslyze or https://github.com/rbsec/sslscan

{% endhint %}

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Use TLS version `1.2+` with all other protocols disabled.
- Use only well-known and up-to-date TLS libraries.
- Do **not** use weak TLS Cipher Suites.

<details>
<summary>Clarification</summary>

Enable only GCM cipher suites.

You can use [https://ssl-config.mozilla.org](https://ssl-config.mozilla.org/) to select the software you are using and create a configuration file that is secure and compatible with a wide variety of browser versions and server software.
</details>

- If it is necessary to support legacy clients and weak cipher suites, disable at least the following types of cipher suites:

    - Null cipher suite.
    - Anonymous cipher suite.
    - EXPORT cipher suite.

- If it is necessary to support legacy clients and weak cipher suites, enable the [TLS_FALLBACK_SCSV](https://www.rfc-editor.org/rfc/rfc7507) extension to prevent downgrade attacks against clients.
- Disable TLS compression to protect against the [CRIME](https://threatpost.com/crime-attack-uses-compression-ratio-tls-requests-side-channel-hijack-secure-sessions-091312/77006/) vulnerability which could potentially be used to recover sensitive information such as session cookies.
- Use sufficiently secure Diffie-Hellman parameters (at least 2048 bits) for cipher suites that use the ephemeral Diffie-Hellman key exchange (signified by the `DHE` or `EDH` strings in the cipher suite name).

    You can find guidance on how various web servers can be configured to use these generated parameters at the [Weak DH](https://weakdh.org/sysadmin.html) website.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Use TLS version `1.3` with all other protocols disabled.

# Certificates

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Use only cryptographically strong algorithms to create a signature, see the [Cryptography: Encryption](/Web%20Application/Cryptography/Encryption/README.md) page.
- Generate keys of the proper size to provide the desired level of security, see the [Cryptography: Encryption](/Web%20Application/Cryptography/Encryption/README.md) page.
- Comply with requirements from the [Cryptography: Cryptographic Keys Management](/Web%20Application/Cryptography/Cryptographic%20Keys%20Management/README.md) page.
- Use strong cryptographic hashing algorithms such as hashing algorithms from the `SHA-2` or `SHA-3` family, see the [Cryptography: Hashing](/Web%20Application/Cryptography/Hashing/README.md) page.
- Use the [fully qualified domain name (FQDN)](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) of the server as the domain name (or subject) of the certificate.
- Set the primary [fully qualified domain name (FQDN)](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) to the `commonName (CN)` attribute and the full list of [fully qualified domain names (FQDNs)](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) in the `subjectAlternativeName (SAN)` attribute.
- Include the `www` subdomain to a certificate.
- Do **not** include unqualified domain names in a certificate.

<details>
<summary>Clarification</summary>

A fully qualified hostname is a name that includes a DNS domain name as a suffix, while an unqualified hostname does not have a domain suffix. So, for example, `api.website.local` is a fully qualified DNS name, while `api` is an unqualified name.
</details>

- Do **not** include IP addresses in a certificate.
- Do **not** include internal domain names in externally facing certificates. If a server is accessible using both internal and external [fully qualified domain names (FQDNs)](https://en.wikipedia.org/wiki/Fully_qualified_domain_name), configure it with multiple certificates.


- Avoid using wildcard certifications, for example, `*.website.local`. Only use wildcard certificates where you really need to, and not for convenience.

<details>
<summary>Clarification</summary>

When multiple systems share a wildcard certificate, the likelihood that the private key for the certificate has been compromised increases because the key may be present on multiple systems. Additionally, the value of such a key is significantly increased, making it a more attractive target for attackers.
</details>

-  Do **not** use wildcard certificates for systems at different trust levels.

<details>
<summary>Clarification</summary>

**The same trust level**

- Two VPN gateways could use a shared wildcard certificate.
- Multiple instances of a web application could share a certificate.

**Different trust levels**

- A VPN gateway and a public webserver should not share a wildcard certificate.
- A public web server and an internal server should not share a wildcard certificate.
</details>

- If wildcard certificates are used:

    - Limit the scope of a wildcard certificate by issuing it for a subdomain such as `*.foo.bar.website.local`.
    - Consider using a reverse proxy that does TLS termination so that the wildcard private key is only present on one system.
    - Maintain all systems that share the same wildcard certificate so that they can all be updated if the certificate expires or is compromised.

- Use Certification Authority Authorization (CAA) DNS records to define which CAs are permitted to issue certificates for a domain.

<details>
<summary>Clarification</summary>

Certification Authority Authorization (CAA) DNS records contain a list of CAs, and any CA who is not included in that list should refuse to issue a certificate for a domain. This can help to prevent an attacker from obtaining unauthorized certificates for a domain through a less-reputable CA.

Where it is applied to all subdomains, it can also be useful from an administrative perspective by limiting which CAs administrators or developers are able to use, and by preventing them from obtaining unauthorized wildcard certificates.
</details>

- Provide the full certificate chain.

<details>
<summary>Clarification</summary>

If a user does not know or trust intermediate CAs, the certificate validation will fail, even if a user trusts an ultimate root CA. Since a user can not establish a chain of trust between a certificate and the root, the validation will fail. To avoid this, any intermediate certificates should be provided alongside the main certificate.
</details>

# Content caching

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Disable caching for `GET`, `HEAD` and `POST` requests that pass sensitive data. Add the following HTTP header to disable caching for such requests:

    ```
    Cache-Control: no-store, no-cache, must-revalidate
    ```

# Secure HTTP headers

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Add HTTP `Strict-Transport-Security` response header to protect a user from downgrade attacks:

    ```
    Strict-Transport-Security: max-age=31622400; includeSubDomains; preload
    ```

# References

- [OWASP Cheat Sheets: Transport Layer Protection Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html)
