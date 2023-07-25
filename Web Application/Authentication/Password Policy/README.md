# Overview

This page contains a recommended example of a password policy.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Password length from 12 characters to 64 characters.
- Use user-provided passwords as is. Do **not** truncate spaces or other special characters.
- Prevent the use of passwords that contains dictionary words and compromised passwords during registration, login, and password change.
- Create a wordlist of breached passwords and check passwords against it. You can use the following wordlists to create a custom one:

    - [https://github.com/danielmiessler/SecLists](https://github.com/danielmiessler/SecLists)
    - [https://github.com/Dormidera/WordList-Compendium](https://github.com/Dormidera/WordList-Compendium)
    - [https://github.com/ignis-sec/Pwdb-Public](https://github.com/ignis-sec/Pwdb-Public)
    - [https://github.com/ihebski/DefaultCreds-cheat-sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet)

- Do **not** use password composition rules limiting the type of characters.
- Do **not** set requirements for upper or lower case, numbers or special characters.
- Do **not** use periodic credential rotation or password history requirements.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Use a third-party service (that provides a zero-knowledge proof API) to validate passwords against breached ones.

    - Make sure plain text passwords are not sent or used in verifying the breach status of a password.

- Allow using Unicode characters in passwords. Consider a single Unicode code as a character. In other words, 12 emoji should be a valid password.
- Notify users if they use a breached password during sign-in. For example, you can send a notification email with the link to the password change page.
