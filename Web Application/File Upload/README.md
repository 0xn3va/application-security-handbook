# Overview

This page contains recommendations for the implementation of secure file upload functionality.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Process files with well-known and up-to-date libraries and frameworks.
- Implement a comprehensive validation of each uploaded file, which must include at least:

    - [File name sanitization](#file-name-sanitization)
    - [File extension validation](#file-extension-validation)
    - [Content-Type validation](#content-type-validation)

- Follow file storage best practices, see the [File storage](#file-storage) section.
- Set file size limits and implement upload rate limits, see the [Size and rate limits](#size-and-rate-limits) section.
- Log successful and unsuccessful file upload attempts and access to files events, see the [Logging and Monitoring](/Web%20Application/Logging%20and%20Monitoring/README.md) page.
- Implement Content Disarm and Reconstruction (CDR) for [potentially dangerous file types](#dangerous-file-types), see the [Content Disarm and Reconstruction (CDR)](#content-disarm-and-reconstruction-cdr) section.

# File name sanitization

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Implement a comprehensive input validation for file names, see the [Input Validation](/Web%20Application/Input%20Validation/README.md) page.

    - Enforce a limit to file name. For example, require the length of a file name to be between `1` and `64` characters.
    - Define an allow-list of characters for file names. The allow list must consist of alphanumeric characters, hyphens, spaces, and period characters.

<details>
<summary>Example</summary>

The regex `\A[A-Za-z0-9._-]{1,64}\z` can be used to validate that a file name only includes alphanumeric, hyphen, space and period characters within a minimum of 1 until a maximum of 64 characters length.
</details>

# File extension validation

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Implement a comprehensive input validation for file extensions, see the [Input Validation](/Web%20Application/Input%20Validation/README.md) page.

    - Define an allow list of extensions. Do **not** use block list validation.

<details>
<summary>Clarification</summary>

Unfortunately, block list validation may miss unknown bad values that an attacker could leverage to bypass the validation. For example, mixing lowercase and uppercase characters; adding a valid extension before or after an invalid extension; adding special characters within or at the end of the extension; or a combination of the previous bypassing techniques.

```
file.pHp
file.png.php
file.php.png
file.php%20
file.php%00
file.php%00.png
file.png.jpg.php
file.PhP%00.pNg%00.jpG
file.%E2%80%AEphp.jpg
```

**Note** that input data **must be normalized** before any comparison, validation or processing. If normalization is not done properly, allow list validation can be bypassed easily, check the [Normalization section in the Input Validation](/Web%20Application/Input%20Validation/README.md) page.
</details>

- Avoid uploading files with potentially dangerous extensions, see the [Dangerous file type](#dangerous-file-types) section.

# Content-Type validation

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Define an allow list of `MIME types` and validate the `Content-Type` HTTP header against this list. Do **not** use a block list validation.

<details>
<summary>Note</summary>

The browser sends the following `Content-Type` header for each uploaded file in `form-data` requests:

```
POST /photo/upload HTTP/1.1
Content-Length: 1337
Content-Type: multipart/form-data; boundary=---------------------------974767299852498929531610575

-----------------------------974767299852498929531610575
Content-Disposition: form-data; name="description"

some text
-----------------------------974767299852498929531610575
Content-Disposition: form-data; name="myFile"; filename="foo.txt"
Content-Type: text/plain

(content of the uploaded file foo.txt)
-----------------------------974767299852498929531610575--
```

This `Content-Type` header is often provided as MIME-type by language/frameworks.
</details>

- Implement [MIME Sniffing](#mime-sniffing-implementation) to extract the effective MIME type of a file's content.
- Avoid uploading files with potentially dangerous MIME types, see the [Dangerous file types](#dangerous-file-types) section.

## MIME Sniffing implementation

The `Content-Type` header can be easily manipulated by a user and spoofed to bypass `Content-Type` verifications. Therefore, it can not be fully trusted. However, there is a content sniffing algorithm known as [MIME Sniffing](https://mimesniff.spec.whatwg.org/) that provides a more in-depth `Content-Type` validation and can be used as a workaround for this issue.

The following libraries can be used to implement MIME sniffing:

{% tabs %}
{% tab title="C/C++" %}

[libmagic](https://man7.org/linux/man-pages/man3/magic_load.3.html) library
{% endtab %}

{% tab title="Go" %}

Function [DetectContentType()](https://pkg.go.dev/net/http#DetectContentType) at [net/http](https://pkg.go.dev/net/http) package
{% endtab %}

{% tab title="Node.js" %}

[mmmagic](https://github.com/mscdex/mmmagic) library
{% endtab %}

{% tab title="Python" %}

[python-magic](https://github.com/ahupp/python-magic) wrapper for [libmagic](https://man7.org/linux/man-pages/man3/magic_load.3.html)
{% endtab %}
{% endtabs %}

# File storage

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Use Universal Unique Identifier (UUID) as a name for uploaded files, see the [Universal Unique Identifier (UUID)](/Web%20Application/Cryptography/Universal%20Unique%20Identifier%20(UUID)/README.md) page.

<details>
<summary>Clarification</summary>

Using UUID as a file name for uploaded files is the best way to avoid vulnerabilities associated with arbitrary file names.
</details>

- Store uploaded files outside of the webroot folder.
- Store uploaded files separate from application content and code. For example, use a separate volume or a third-party storage.
- Define an appropriate directory structure accordingly to your file system.

<details>
<summary>Clarification</summary>

Most file systems have theoretical and practical limits on the amount of files/directories that can be stored within a single directory or file system. Moreover, there are some performance issues when a single directory contains a huge amount of files. 

Therefore, it is important to be aware of file system limits and create a directory structure that mitigates these issues.

There is no standard for creating such a directory structure, but a good approach that distributes files within a directory structure is as the following:

1. Take the first two characters from the UUID file name.
1. If does not exist, create a directory with those two characters as the folder name.
1. Store the file within the directory named with those two characters from its UUID.

The following represents the directory structure result of using this approach:

```
FileStorage/
    18/
        1859b296-8bc3-4612-8179-8bec1dcaac16
    1f/
        1f663aa5-cef7-4f6a-85b2-48294de0cbcc
    97/
        973eb004-44ec-4900-8109-240076b649ba
    e9/
        e9bd6269-ef70-49d8-a73d-5781c30e89a9
```
</details>

- Set up file storage and uploaded files with the minimum necessary set of permissions according to the purpose of use.

<details>
<summary>Example</summary>

- Application users and groups have read and write permissions to store files.
- Administrative accounts (backup or monitoring) have read-only permissions.
- Any other users do not have access to the file storage.
- If files should not be modified, set permissions to read-only.
- If files can be modified by an application (on a user's update request), set permissions for writing only to the application users and groups.
</details>

- Reset permissions for all uploaded files.
- Make sure the storage is encrypted at rest.
- Make sure there is a backup strategy for the storage.
- Make sure there is storage capacity monitoring.

<details>
<summary>Clarification</summary>

Storage capacity must be monitored to avoid reaching its maximum capacity and leading to a denial of service.
</details>

- If file execution is needed, the file must be scanned with an anti-malware solution and executed only within a separate and sandboxed environment.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Use third-party services to manage file-uploading functionality.

<details>
<summary>Clarification</summary>

There are third-party systems that can handle the file-uploading functionality and provide enterprise-grade functionality and security. Some of the most popular are:

- [Filestack](https://www.filestack.com/)
- [Transloadit](https://transloadit.com/)
- [Cloudinary](https://cloudinary.com/documentation/upload_widget)
- [Uploadcare](https://uploadcare.com/)
</details>

- Use a separate domain/service to store/host uploaded files.

<details>
<summary>Example</summary>

If the main domain is `website.local`, use a separate domain such as `user-content.website-static.local` to store uploaded files.
</details>

- Enforce a strict Content Security Policy to the service that hosts uploaded files, see the [Content Security Policy (CSP)](/Web%20Application/Content%20Security%20Policy%20(CSP)/README.md) page.
- Scan every uploaded file with an anti-malware solution such as [VirusTotal](https://www.virustotal.com).
- If files need to be modified by an application (on the user's update request as an example), set up versioning on files.

# Authorization

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Allow file uploads **only** for authenticated users.
- Validate a user has the necessary permissions to upload files before uploading them.
- Comply with the requirements from the [Authorization](/Web%20Application/Authorization/README.md) page.
- Implement protection against Cross-Site Request Forgery (CSRF) attacks for file upload functionality, see the [Vulnerability Mitigation: Cross-Site Request Forgery (CSRF)](/Web%20Application/Vulnerability%20Mitigation/Cross-Site%20Request%20Forgery%20(CSRF)/README.md) page.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Allow access to uploaded files **only** for registered users.
- Do **not** expose uploaded files to anonymous users. 

# Size and rate limits

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Set a limit on the size of the uploaded file and validate that the file size does not exceed the specified limit before handling it.

    - Use the `Content-Length` header to identify a file size.
    - Do **not** rely on user-controlled parameters such as form values to identify a file size.
    - Do **not** rely on [chunked transfer encoding](https://en.wikipedia.org/wiki/Chunked_transfer_encoding) to upload files.

<details>
<summary>Clarification</summary>

If no restriction on maximum file size is established, an attacker can submit huge files to fill up the storage's quota and lead to a denial of service on the system.

You should not rely on a user-controlled parameter such as `GET`, `POST`, or `PUT` parameters, because an attacker can easily bypass size validation using that parameter to set an arbitrary size and upload bigger files than allowed.

Using chunked transfer encoding is also not recommended, because there is no way to identify a file size. Moreover, there is [a class of vulnerabilities](https://portswigger.net/web-security/request-smuggling) that abuse differences in parsing requests with `Content-Length` and `Transfer-Encoding` headers to smuggle HTTP requests.
</details>

- Limit the number of uploaded files per each request.
- Limit the amount of file upload requests (single or multiple upload file requests) a single user can perform within a reasonable fixed period of time according to an application and user needs.

<details>
<summary>Clarification</summary>

The limit must be set according to an application's requirements and common usage.

For instance:

- A photo album application could allow 100 photo file uploads within 2 minutes.
- A ticketing tracking system could allow 5 files per minute.
</details>

# Content Disarm and Reconstruction (CDR)

Many dangerous file types can be used to attack an application and pose risks for the entity that receives and stores such files and users that access them. Content Disarm and Reconstruction (CDR) techniques allow you to mitigate these risks.

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Implement CDR on images before any further processing, see the [CDR Images](#cdr-images) section.
- Implement CDR on multimedia files before any further processing, see the [CDR Multimedia](#cdr-multimedia-files) files section.
- Implement CDR on XML and HTML files before any further processing, see the [CDR Content and Markup Languages](#cdr-content-and-markup-languages) section.
- Implement CDR on compressed files before any further processing, see the [CDR Compressed files](#cdr-compressed-files) section.
- Implement CDR on Microsoft Office documents and PDFs before any further processing, see the [CDR Documents](#cdr-documents) section.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Use a third-party solution to perform CDR on known dangerous files before any further processing.

## CDR Images

Image uploading may lead to different security threats like the following ones:

- Embedding malicious code into metadata.
- Cross-Site Scripting or XML external entity (XXE) injection through SVG images.
- Memory leaks due to image error processing.
- Remote code execution due to improper image processing.

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Create an allow list of image formats and work only with images of these formats.

    - Avoid processing `SVG` files.

- If it is necessary to work with `SVG` files implement extra protection layers to reduce the blast radius:

    - Implement SVG sanitization, see [DOMPurify](https://github.com/cure53/DOMPurify).
    - Host images on a separate domain, see the [File storage](#file-storage) section.
    - Enforce a strict Content Security Policy to that domain, see the [Content Security Policy (CSP)](/Web%20Application/Content%20Security%20Policy%20(CSP)/README.md) page.

- Use well-known and up-to-date libraries to process images.
- Process images (cropping, resizing, etc.) in a sandboxed environment.

<details>
<summary>Clarification</summary>

Most imaging libraries are quite complex solutions that support dozens of different file types. Often it leads to high severe vulnerabilities. Since many of these libraries are written in low-level languages, this provides an attacker with additional opportunities for remote code execution and accessing arbitrary memory regions. Experience has shown that we can not trust such libraries. They should be isolated to reduce the damage radius in case of exploitation.

Process images in a sandboxed environment (such as an isolated container or a third-party system) to mitigate exploitation of vulnerabilities in used libraries.
</details>

- Remove metadata from images such as EXIF metadata.

<details>
<summary>Clarification</summary>

Metadata may contain Personal Identifiable Information (PII), for example:

- Date and time.
- Geo-localization.
- Camera or phone model and settings.
- etc.

That information will be available to all users, including attackers, who can access such images.

Moreover, metadata can be used by an attacker to deliver malicious payloads.
</details>

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Insert random noise into image content, see the [Random Generators](/Web%20Application/Cryptography/Random%20Generators/README.md) page.

<details>
<summary>Example</summary>

You can add or subtract 1 from each RGB byte. This will add subtle noise to the image output but will make the output unpredictable for an attacker to leverage it for exploitation.
</details>

## CDR Multimedia files

Multimedia file uploading may lead to different security threats like the following ones:

- Memory leaks due to multimedia file error processing.
- Remote code execution due to improper multimedia file processing.
- Server-side Request Forgery due to improper multimedia file processing.

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Use well-known and up-to-date libraries to process audio and video files.
- Process audio and video files in a sandboxed environment.

<details>
<summary>Clarification</summary>

The security threats for processing audio and video files are similar to those described in the [CDR Images](#cdr-images) section.

Process audio and video files in a sandboxed environment (such as an isolated container or a third-party system) to mitigate exploitation of vulnerabilities in used libraries.
</details>

## CDR Content and Markup Languages

XML and HTML file uploading may lead to different security threats like the following ones:

- Arbitrary JavaScript execution via malicious HTML.
- Arbitrary JavaScript execution via malicious CSS.
- XML External Entity injection via malicious XML.

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Avoid processing `HTML` files.
- If it is necessary to work with `HTML` files implement extra protection layers to reduce the blast radius:

    - Implement HTML sanitization, see [DOMPurify](https://github.com/cure53/DOMPurify).
    - Use a separate domain to store/host uploaded HTML files, see the [File storage](#file-storage) section.
    - Enforce strict Content Security Policy to that domain, see the [Content Security Policy (CSP)](/Web%20Application/Content%20Security%20Policy%20(CSP)/README.md) page.

- Comply with the requirements from the [Vulnerability Mitigation: XML External Entity (XXE)](/Web%20Application/Vulnerability%20Mitigation/XML%20External%20Entity%20(XXE)%20Injection/README.md) page for XML files.

## CDR Compressed files

Compressed file uploading may lead to different security threats like the following ones:

- Denial of service due to unpacking a Zip bomb file.
- Path traversal during unpacking compressed files leads to overwriting existing files.
- Accessing restricted files due to improper handling of symbolic links in compressed files.

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Verify decompress output size before unpacking files.
- Remove symbolic links from the unpacked files.
- Reset permissions for all unpacked files.
- Make sure unpacking does not overwrite existing files.
- Validate each unpacked file according to the requirements on this page.
- Comply with requirements from the [Vulnerability Mitigation: Path Traversal](/Web%20Application/Vulnerability%20Mitigation/Path%20Traversal/README.md) page.

## CDR Documents

Microsoft and PDF document uploading may lead to different security threats like the following ones:

- Execution of arbitrary JavaScript embedded into a PDF document.
- Embedding malicious files to a PDF document.
- Remote code execution due to improper PDF processing.
- Malicious macros, Flash, OLE objects or HTA Handlers in Microsoft documents (Word, Excel or PowerPoint).

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Scan Microsoft and PDF documents with an anti-malware solution such as [VirusTotal](https://www.virustotal.com).

# Dangerous file types

Processing some types of files is potentially dangerous because it can easily lead to a vulnerability. It is recommended to avoid processing such files and block the upload of any files with the appropriate extensions. If it is necessary to accept any of these files, it is highly recommended to handle such files in a sandbox, implement extra layer protections (like Content Security Policy) and perform Content Disarm and Reconstruction (CDR) to reduce the blast radius.

Below you can find a list of potentially dangerous files. This list is not exhaustive.

{% tabs %}
{% tab title="Configuration" %}

**Extensions**

`.config`, `.ini`, `.htaccess`, `.htpasswd`, `.xml`
{% endtab %}

{% tabs %}
{% tab title="Compressed" %}

**Extensions**

`.zip`, `.rar`, `.tar`, `tgz`, `.tar.gz`

**MIME Types**

- `application/gzip`
- `application/zip`
- `application/vnd.rar`
- `application/x-tar`
{% endtab %}

{% tab title="Executable" %}

**Extensions**

`.bat`, `.cmd`, `.com`, `.cpl`, `.csh`, `.dll`, `.exe`, `.hta`, `.inf`, `.jar`, `.js`, `.msi`, `.msc`, `.msp`, `.mpkg`, `.ps1`, `.ps1xml`, `.ps2`, `.ps2xml`, `.psc1`, `.psc2`, `.sh`, `.swf`, `.vb`, `.vbe`, `.vbs`, `.ws`, `.wsf`, `.wsc`, `.wsh` 

**MIME Types**

- `application/vnd.microsoft.portableexecutable`
- `application/java-archive`
- `application/x-csh`
- `application/vnd.apple.installer+xml`
- `application/x-sh`
- `application/x-shellscript`
- `application/x-msdownload`
- `application/x-shockwave-flash`
- `text/javascript`

{% endtab %}

{% tab title="Images" %}

**Extensions**

`.svg`

**MIME Types**

`image/svg+xml`
{% endtab %}

{% tab title="Multimedia" %}

**Extensions**

`.aac`, `.avi`, `.mp3`, `.mp4`, `.mpeg`, `.oga`, `.ogv`, `.wav`

**MIME Types**

- `audio/aac`
- `audio/mpeg`
- `audio/ogg`
- `video/x-msvideo`
- `video/mp4`
- `video/mpeg`
- `video/ogg`
- `audio/wav`
{% endtab %}

{% tab title="Server-side scripting" %}

**Extensions**

- ASP `.asp`, `.aspx`, `.config`, `.ashx`, `.asmx`, `.aspq`, `.axd`, `.cshtm`, `.cshtml`, `.rem`, `.soap`, `.vbhtm`, `.vbhtml`, `.asa`, `.cer`, `.shtml`
- Coldfusion `.cfm`, `.cfml`, `.cfc`, `.dbm`
- Erlang Yaws Web Server `.yaws`
- Flash `.swf`
- JSP `.jsp`, `.jspx`, `.jsw`, `.jsv`, `.jspf`, `.wss`, `.do`, `.action`
- Perl `.pl`, `.cgi`
- PHP `.php`, `.php2`, `.php3`, `.php4`, `.php5`, `.php6`, `.php7`, `.phps`, `.phps`, `.pht`, `.phtm`, `.phtml`, `.pgif`, `.shtml`, `.htaccess`, `.phar`, `.inc`
- Python `.py`
- XML `.xml`

**MIME Types**

- `application/x-httpd-php`
- `application/xml`
- `text/xml`
- `text/x-python`
- `application/x-python-code`
{% endtab %}
{% endtabs %}
