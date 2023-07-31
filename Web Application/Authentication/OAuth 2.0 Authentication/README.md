# Overview

This page contains recommendations for the implementation of OAuth 2.0 authentication.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Do **not** implement support of OAuth2 authentication from scratch. Instead, use well know and up-to-date frameworks or packages.
- Use the Authorization Grant Flow to implement OAuth2 authentication.
- Do **not** use the Implicit Grant Flow. Disable the use of the Implicit Grant Flow if a framework or package supports the flow.
- Use URLs from an allow list as `redirect_uri`.
- Do **not** use wildcards for defining `redirect_uri`.
- Use a unique allow list with `redirect_uri` for each OAuth2 client.
- Implement CSRF protection using the `state` parameter, see the [Vulnerability Mitigation: Cross-Site Request Forgery (CSRF)](/Web%20Application/Vulnerability%20Mitigation/Cross-Site%20Request%20Forgery%20(CSRF)/README.md) page.
- Generate a unique `state` parameter for each authentication attempt.
- Generate the `state` parameter using a cryptographically strong random generator, see the [Cryptography: Random Generators](/Web%20Application/Cryptography/Random%20Generators/README.md) page.
- Use the `state` parameter of length 16+ bytes.
- Generate a unique `authorization code` for each authentication attempt.
- Generate the `authorization code` using a cryptographically strong random generator, see the [Cryptography: Random Generators](/Web%20Application/Cryptography/Random%20Generators/README.md) page.
- Use `authorization codes` of length 16+ bytes.
- Set expiration time for an `authorization code` < 1 hour.
- Use an authorization code once. Delete an authorization code or transfer it to a final status that prohibits reusing.
- Do **not** redirect users to URLs specified in the parameters without checking them against an allow list.
- Do **not** assign OAuth authentication for already existing accounts during authentication.

<details>
<summary>Clarification</summary>

If an application automatically assigns OAuth authentication to an existing account during authentication, this means that an attacker could potentially take over a victim's account using the flow like this:

1. An attacker creates an account in a service provider using a victim's email linked to an account in our application.
1. An attacker uses `Sign in with ...` to login into an account in our application.
1. An application will find a victim's account using an email address from an attacker's service provider account (an attacker linked that email with a service provider account).
1. Since the emails are the same, an application will log in an attacker to a victim's account.
</details>

- Comply with requirements from the [Error and Exception Handling](/Web%20Application/Error%20and%20Exception%20Handling/README.md) page.
- Log all authentication decisions (successful and not successful), see the [Logging and Monitoring](/Web%20Application/Logging%20and%20Monitoring/README.md) page.

    - Successful authentication
    - Failed authentication
    - Successful connection to an OAuth provider
    - Failed connection to an OAuth provider
    - Disconnect an OAuth provider

- Comply with requirements from the [Sensitive Data Management](/Web%20Application/Sensitive%20Data%20Management/README.md) page.
- Limit the number of attempts to sign in for a certain period, see the [Vulnerability Mitigation: Brute-force](/Web%20Application/Vulnerability%20Mitigation/Brute-force/README.md) page.
- Enforce multi-factor authentication, see the [Authentication: Multi-factor Authentication](/Web%20Application/Authentication/Multi-factor%20Authentication/README.md) page.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Notify a user via an available communication channel (email, push, SMS, etc.) about successful login under their account from an unknown location, browser, client, etc.

# Frameworks & packages for implementation of OAuth2 authentication

{% tabs %}
{% tab title="Go" %}

Use the [oauth2](https://pkg.go.dev/golang.org/x/oauth2) package, that implements an OAuth2 client.

```go
import (
    "crypto/rand"
    "context"
    "encoding/base64"
    "golang.org/x/oauth2"
    "fmt"
    "log"
    "os"
    "net/http"
    "io/ioutil"
)

var endpoint = oauth2.Endpoint{
    AuthURL: "https://website.local/auth",
    TokenURL: "https://website.local/oauth/access_token",
    AuthStyle: oauth2.AuthStyleInParams,
}

var oauthConfig = &oauth2.Config{
    RedirectURL: "http://localhost:8000/auth/callback",
    ClientID: os.Getenv("OAUTH_CLIENT_ID"),
    ClientSecret: os.Getenv("OAUTH_CLIENT_SECRET"),
    Scopes: []string{"user:email"},
    Endpoint: endpoint,
}

// initiate an authentication process
func oauthLogin(w http.ResponseWriter, r *http.Request) {
    state := generateState()
    u := oauthConfig.AuthCodeURL(state)
    http.Redirect(w, r, u, http.StatusTemporaryRedirect)
}

// exchange code for access token
// use an access token to make requests to a resource server
func oauthCallback(w http.ResponseWriter, r *http.Request) {
    code := r.FormValue("code")
    token, err := oauthConfig.Exchange(context.Background(), code)
    if err != nil {
        log.Println(fmt.Errorf("code exchange wrong: %s", err.Error()))
        http.Redirect(w, r, "/", http.StatusTemporaryRedirect)
        return
    }
    data, err := getUserData(token)
    if err != nil {
        log.Println(err.Error())
        http.Redirect(w, r, "/", http.StatusTemporaryRedirect)
        return
    }
    fmt.Fprintf(w, "UserInfo: %s\n", data)
}

func generateState() string {
    b := make([]byte, 32)
    rand.Read(b)
    state := base64.URLEncoding.EncodeToString(b)
    return state
}

func getUserData(oauth2.Token *token) {
    url := "https://api.resource.server.local/user"
    req, err := http.NewRequest("GET", url, nil)
    req.Header.Add("Authorization", "Bearer " + token.AccessToken)
    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        return nil, fmt.Errorf("failed getting user data: %s", err.Error())
    }
    defer resp.Body.Close()
    body, err := ioutil.ReadAll(resp.Body)
    if err != nil {
        return nil, fmt.Errorf("failed read response: %s", err.Error())
    }
    return body, nil
}
```
{% endtab %}
{% endtabs %}

# References

- [OWASP Cheat Sheet Series: Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html#logging-and-monitoring)
