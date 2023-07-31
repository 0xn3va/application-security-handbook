# Overview

This page contains recommendations for using regular expressions.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Do **not** use regular expressions if there is a clean non-regex solution, for example, searching for a substring or using `if` conditions.
- Use regular expression engines that provide linear time expression matching at least for user-provided regular expressions or matching "hard-coded" expressions against user-controlled data, see the [Linear time regular expression matching implementation](#linear-time-regular-expression-matching-implementation) section.

<details>
<summary>Clarification</summary>

Many regex engines support backtracking that causes them to work very slowly in some cases (exponentially related to input size), see the [Vulnerability Mitigation: Regular Expression Denial of Service (ReDoS)](/Web%20Application/Vulnerability%20Mitigation/Regular%20Expression%20Denial%20of%20Service%20(ReDoS)/README.md) page.
</details>

- Do **not** use `multi-line` matching mode in regexes that are used for validation. Otherwise, make sure that full string matching `^...$` works as expected or rewrite regexes using more specific expressions like `\A...\z`.

    - Remember in some engines `multi-line` matching mode is a default mode, for example, the built-in regex engine in Ruby.

<details>
<summary>Clarification</summary>

In `multi-line` mode, the expressions with `^` and `$` are matched differently. For example, `$` matches not only before the end of the string but also at the end of each line. So, if there is a validation that uses a regular expression in `multi-line` mode, an attacker can use a new line `\x0a` to bypass this validation. Consider a regular expression matching in Python from the snippet below.

```python
import re

p = re.compile(r'^\d{1,3}$')

p.match('137') is not None
# => True
p.match('1337') is not None
# => False
p.match('abc') is not None
# => False
p.match('137\nabc') is not None
# => False
```

The regex from the snippet matches full strings containing numbers from 1 to 3 digits long. However, enabling `multi-line` mode completely changes this behaviour.

```python
import re

p = re.compile(r'^\d{1,3}$', re.MULTILINE)

p.match('137') is not None
# => True
p.match('1337') is not None
# => False
p.match('abc') is not None
# => False
p.match('137\nabc') is not None
# => True
```

As can be seen, in `multi-line` mode the string `137\nabc` will be successfully matched. To avoid this behaviour, disable `multi-line` mode (this is the preferred solution) or rewrite the regex using `\A` and `\Z`:

```python
import re

# preferred
p = re.compile(r'^\d{1,3}$')

p.match('137\nabc') is not None
# => False

# or
p = re.compile(r'\A\d{1,3}\Z', re.MULTILINE)

p.match('137\nabc') is not None
# => False
```
</details>

- Implement input validation for strings for matching, at least for string length and allowed characters, see the [Input Validation](/Web%20Application/Input%20Validation/README.md) page.
- Use the following practices to simplify regular expressions and reduce the likelihood of problems with catastrophic backtracking:

    - Avoid nested quantifiers, for example `(a+)+`.
    - Try to be as precise as possible and avoid the `.` pattern.
    - Use reasonable ranges, for example `{1,10}`, for repeating patterns instead of unbounded `*` and `+` patterns.
    - Simplify character ranges, for example `[ab]` instead of `[a-z0-9]`.

<details>
<summary>Detecting Catastrophic backtracking</summary>

You can use [doyensec/regexploit](https://github.com/doyensec/regexploit) to detect Catastrophic backtracking in your regexes that lead to ReDoS.

**regexploit does not guarantee the detection of 100% vulnerable regexes, this is just one of the relatively easy ways to check your regex**

```bash
$ python3 -m venv .env
$ source .env/bin/activate
$ pip install regexploit
$ regexploit
v\w*_\w*_\w*$
Pattern: v\w*_\w*_\w*$
---
Worst-case complexity: 3 ⭐⭐⭐ (cubic)
Repeated character: [5f:_]
Final character to cause backtracking: [^WORD]
Example: 'v' + '_' * 3456 + '!'
```
</details>

- Log regex failures, especially if a regex is used for validation, see the [Logging and Monitoring](/Web%20Application/Logging%20and%20Monitoring/README.md) page.
- Comply with requirements from the [Error and Exception Handling](/Web%20Application/Error%20and%20Exception%20Handling/README.md) page.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Use regular expression engines that provide linear time expression matching for matching all regular expressions, see the [Linear time regular expression matching implementation](#linear-time-regular-expression-matching-implementation) section.

# Linear time regular expression matching implementation

There is the [re2](https://github.com/google/re2) engine that provides linear time expression matching. Try to find a library that is based on the [re2](https://github.com/google/re2) engine.

{% tabs %}
{% tab title="Go" %}

Use the [regexp](https://pkg.go.dev/regexp) package that uses the [re2](https://github.com/google/re2) engine.

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    inputData := "some text to match"
    match, err := regexp.MatchString("[a-z]{1,16}", inputData)
    if err == nil {
        fmt.Println("Match:", match)
    } else {
        fmt.Println("Error:", err)
    }
}
```
{% endtab %}

{% tab title="Java" %}

Use the [re2j](https://github.com/google/re2j) package which is a port of C++ [re2](https://github.com/google/re2) to pure Java.

```java
import com.google.re2j.Matcher;
import com.google.re2j.Pattern;

Pattern p = Pattern.compile("[a-z]{1,16}");
Matcher m = p.matcher("some text to match");
assertTrue(m.find());
```
{% endtab %}

{% tab title="Node.js" %}

Use the [node-re2](https://www.npmjs.com/package/re2) package which is a wrapper for the [re2](https://github.com/google/re2) engine.

```javascript
var RE2 = require("re2");
var re = new RE2("[a-z]{1,16}");
var result = re.exec("some text to match");
console.log(result);
```
{% endtab %}

{% tab title="Python" %}

Use the [google-re2](https://pypi.org/project/google-re2/) package which is a wrapper for the [re2](https://github.com/google/re2) engine.

```python
import re2

re2.compile('[a-z]{1,16}')
print(p.match('some text to match').string)
```
{% endtab %}
{% endtabs %}

# References

- [GitLab Docs: Secure coding development guidelines - Regular Expressions guidelines](https://docs.gitlab.com/ee/development/secure_coding_guidelines.html#regular-expressions-guidelines)
