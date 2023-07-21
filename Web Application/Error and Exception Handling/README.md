# Overview

This section contains recommendations for handling errors and exceptions.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Handle each error and exception using a common handling component.
- Override the default error page for unhandled errors and exceptions.
- Error and exception handling complies with the `fail-secure` security design principle:
    - An application is returned to the most secure state when an error or exception occurs.
    - An application denies access by default when an error or exception occurs.
    - Error messages are not verbose in nature:
        - Do **not** pass sensitive data in error messages see [Sensitive Data Management](/Web%20Application/Sensitive%20Data%20Management/README.md).
        - Do **not** include a stack trace in error messages and HTTP responses.
        - Do **not** pass server configuration information (name, version, etc.) in error messages.

<details>
<summary>Clarification</summary>

The `fail-secure` principle states that raising an error or exception must not lead an application to an insecure state that allows a user to gain additional privileges. It means that a user will not be able to bypass security checks, get additional information or elevate their privileges by causing an error or exception in an application.

Consider the following code snippet that implements an authorization check:

```javascript
isAdmin = true
try {
    codeWhichMayFail()
    isAdmin = isUserInRole("ADMIN")
} catch (error) {
    log.warn(error)
}
return isAdmin
```

As can be seen, admin access is allowed by default. Therefore, if `codeWhichMayFail` raises an exception, `isAdmin` will be set to `true` and the authorization check will be bypassed.

The proper validation will look like this:

```javascript
try {
    codeWhichMayFail()
    isAdmin = isUserInRole("ADMIN")
    return isAdmin
} catch (error) {
    log.warn(error)
    return false
}
```

</details>

- Do **not** ignore errors in security-related components like crypto modules.
- Log errors according to the [Logging and Monitoring](/Web%20Application/Logging%20and%20Monitoring/README.md) section.

# Error and exception handling

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

{% tabs %}
{% tab title="Go" %}

In Go, you have to be aware of uncaught panics and handle them to prevent an application from crashing. You can omit the goroutine stack traces entirely by setting the `GOTRACEBACK` environment variable to `none`. In this case, you will only receive a panic message. See [https://pkg.go.dev/runtime#hdr-Environment_Variables](https://pkg.go.dev/runtime#hdr-Environment_Variables)

```go
package main

func main() {
    panic("oops!")
}
```

```bash
$ go run crash.go
panic: oops!

goroutine 1 [running]:
main.main()
	/tmp.MVY5tcbe/crash.go:4 +0x2c
exit status 2
```

```bash
$ GOTRACEBACK=0 go run crash.go
panic: oops!
exit status 2
```

Additionally, you can override a panic recovery and implement custom logic.

{% hint style="info" %}
Recover only works when called from the same goroutine as the panic is called in, see [https://go.dev/ref/spec#Handling_panics](https://go.dev/ref/spec#Handling_panics)
{% endhint %}

Use the following approaches to implement panic handling:

1. Use the `defer` statement to handle panics, see [https://go.dev/blog/defer-panic-and-recover](https://go.dev/blog/defer-panic-and-recover)

    ```go
    defer func() {
        if err := recover(); err != nil {
            fmt.Printf("recovering from err %v", err)
        }
    }()
    handler(w, r)
    ```

1. Implement a middleware to handle panics.

    ```go
    func panicRecovery(h func(w http.ResponseWriter, r *http.Request)) func(w http.ResponseWriter, r *http.Request) {
        return func(w http.ResponseWriter, r *http.Request) {
            defer func() {
                if err := recover(); err != nil {
                    buf := make([]byte, 2048)
                    n := runtime.Stack(buf, false)
                    buf = buf[:n]
                    fmt.Printf("recovering from err %v\n %s", err, buf)
                    w.Write([]byte(`{"error":"our server got panic"}`))
                }
            }()
            h(w, r)
        }
    }
    http.HandleFunc("/endpoint", panicRecovery(handler))
    ```

Remember that some packages can handle panics out of the box, [net/http](https://pkg.go.dev/net/http) as an example.
{% endtab %}
{% endtabs %}

# References

- [OWASP: Fail Securely](https://owasp.org/www-community/Fail_securely)
- [CWE-636: Not Failing Securely ('Failing Open')](https://cwe.mitre.org/data/definitions/636.html)
