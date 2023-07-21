# Overview

This section contains recommendations for secure processing of sensitive data.

# What is sensitive data?

A list of sensitive data is determined individually for each application. The sensitivity of data is determined during the risk assessment, in which existing assets are evaluated against the CIA triad (Confidentiality, Integrity, Availability). The risk assessment sets the importance of a particular asset for a business.

The following list contains examples of data that are considered sensitive in most cases:

- Password.
- One-time password.
- Session ID.
- Access tokens.
- Personal Identifiable Information or PII (any set of data that could potentially identify a particular person):
    - Name.
    - Surname.
    - Address.
    - Identity document details.
    - Photos and videos by which a person can be identified.
    - Phone number.
    - etc.
- Financial data:
    - Transaction details.
    - Card details.
Payment cardholder data.
    - Bank account details.
    - etc.
- Code words.
- Secret values that allow users to perform some operation, for example, money transferring.
- Database connection strings.
- Encryption keys and other master secrets.
- Commercially-sensitive information:
    - Financial reports.
    - Information about unreleased products or services.
    - etc.
- Information whose collection is illegal in a particular jurisdiction.

# Data storage

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Do **not** send sensitive data to analytics.
- Store sensitive data in plain text **only** in a database. Exceptions:
    - Passwords, store passwords according to the [Password storage](/Web%20Application/Authentication/Authentication%20with%20loging%20and%20password/password-storage.md) section.
    - `cvv2/cvc2` and `PIN` codes of payment cards. This data can not be stored in a database in any form if an application is not PCI DSS compliant.

# Data logging

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Write identifiers of sensitive data, by which they are available in a database, to logs instead of their plain text values.
- Otherwise, mask all logged sensitive data, check out the [Data masking](#data-masking) section below.

# Data masking

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Replace actual values with `n` `"*"` when masking data
- n must be fixed and not depend on the length of the input data.
- You can leave part of the data open if this part is open data or does not allow restoring the actual value. For example, the
first 6 and last 4 digits of a card number are public data that can be left unmasked.

## Example of masking policy

- The card number is masked as `*1234`, where `1234` is the last 4 digits of the card number.
- `cvv2/cvc2` are masqueraded as `***`
- Passwords, tokens, and session IDs are masked with `n` `"*"` characters, e.g. password `****`. `n` must be fixed and independent of the length of an input.
- Name and surname are masked as a minimum in the form `Name S`.
- Phone number is masked as `+1(***)****1234`
- Passport ID is masked as `*12`, where `12` is the last 2 digits of the passport ID.
