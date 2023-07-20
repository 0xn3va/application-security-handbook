# Overview

This page contains requirements for implementing event logging.

Event logging is one of the most important components of application security. Lack of logging or its incorrect implementation may lead to the inability to detect and investigate security incidents.

# General

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Make sure logging is enabled on all components of an application. This includes:

    - Clients such as mobile and browser applications.
    - Application backend.
    - Databases.
    - Other components such as reverse proxies, message brokers, etc.

- Do **not** expose logs outside.
- Do **not** use the debug logging level in production.
- Make sure the default logging level provides enough information for business and security needs.
- Сhanging the logging level must follow the change management processes. For example, through changes in the application configuration before it starts.
- Make sure there is no way intentionally or accidentally disable logging completely. For example, through updating the level in the configuration.
- Define a log format for each event that may include the following:

    - Date and time format.
    - Set of properties.
    - Field lengths.
    - Data types.

    See:

    - [Format of events logged by an application](#format-of-events-logged-by-an-application)
    - [Format of events logged in an access log](#format-of-events-logged-in-an-access-log)

- Make sure compliance with a logging format is guaranteed by a logging module interface at an application source code level.
- Define event classification. Classification can be implemented based on events' type, severity, confidence, responsibility, compliance references, etc.

<details>
<summary>Example of classification by severity</summary>
INFO, DEBUG, WARN, ERROR
</details>

- Make sure there is a process in place for proper monitoring of logged events.

    - Incorporate application logging into any existing log management system or infrastructure, for example, a centralized logging and analysis system.
    - Make sure events are available to appropriate teams.
    - Enable alerting and notify the responsible teams immediately of severe events.
    - Share relevant event information with other detection systems, related organizations and centralized intelligence gathering/sharing systems.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Make a logging module a standard component that can be developed independently and reused across multiple applications.
- Add a logging module to a list of approved and recommended modules.
- Synchronize time across all servers and devices.
- Send logged events to SIEM.

# Logging implementation

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Include the properties of the following categories in a log format: `when`, `where`, `who` and `what`.

    Below you can find examples of properties that can be included in the log format.

{% tabs %}
    {% tab title="when" %}
    - Date and time of logging an event.
    - Date and time of an event (if an event and its logging occurred at different times).
    {% endtab %}

    {% tab title="where" %}
    - Application or component ID.
    - Cluster, hostname, IP address and port number, workstation identity, local device ID.
    - Service, for example, name and protocol.
    - Geolocation.
    - Window/form/page, for example, entry point URL and HTTP method for a web application, dialogue box name.
    - Code location, for example, script name, module name.
    {% endtab %}

    {% tab title="who (human or machine user)" %}
    - User ID e.g. user database table primary key value or user name.
    - User's device/machine ID.
    - User's IP address.
    - Cell/RF tower ID.
    - Mobile number.
    {% endtab %}

    {% tab title="what" %}
    - Type of event.
    - Severity of event, for example fatal, error, warning, info, etc.
    - Flag that indicates that the event is security related.
    - Description.
    {% endtab %}
{% endtabs %}

- Implement **only** Create and Read from the CRUD operations to work with logs.
- Do **not** exclude any events from "known" event sources like penetration testers, auditors, internal systems, "trusted" third-party components or systems, search engine robots, uptime/process and other remote monitoring systems, etc. Add new property in event classification for such events to filter them out later.
- Implement input validation of event data, see the [Input Validation](/Web%20Application/Input%20Validation/README.md) section.
- Implement protection against log injection attacks. Make sure carriage return (CR), line feed (LF) and delimiter characters are sanitized. In other words, an attacker is not able to split a log and add a fake log record.
- If logs are written to a database implement SQL injection protection, see the [SQL Injection (SQLi)](/Web%20Application/Vulnerability%20Mitigation/SQL%20Injection/README.md) section.
- Comply with the requirements from the [Sensitive Data Management](/Web%20Application/Sensitive%20Data%20Management/README.md) section.
- Comply with the requirements from the [Error and Exception Handling](/Web%20Application/Error%20and%20Exception%20Handling/README.md) section.

<div align="left">
<img src="/.gitbook/assets/type-advanced-icon.svg">
</div>

- Do **not** log event data that fail input validation.
- Raise an alert if an event data fails input validation.
- Use separate files or tables to log extended event information such as stack traces, HTTP requests, responses, headers, bodies, etc.

# Log storage

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- If logs are written to the file system:

    - Use a separate partition to write and store logs.
    - Apply strict permissions on which users can access the directories and file permissions within the directories.
    - Grant the minimum required permissions to write logs.

- If the logs are written to a database:

    - Use a separate database account to write logs.
    - Grant very strict database, table, function, and command permissions to this separate database account.
    - The account must have the minimum necessary privileges to write logs.

- Send logs to a logging system isolated from an application, such as a syslog server.
- Use standard formats over **secure** protocols to send event data or log files to other systems. For example, Common Log File System (CLFS) or Common Event Format (CEF) over TLS.
- Store logs in read-only storage.
- Store logs encrypted.
- Keep logs for a certain period following internal requirements and regulatory requirements. For example, keep logs for at least 6 months in fast storage and 24 months in long-term storage.
- Log and monitor all access to logs.
- Limit the number of users who can access logs. Grant them the minimum required permissions.

# What events to log?

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

- Authentication successes and failures.
- Successful and fail password change and reset attempts.
- Authorization (access control) failures.
- Operations requiring elevated privileges.
- Changing critical data, such as email, phone number, username, card data, etc.
- Changing or disabling security mechanisms, for example, disabling two-factor authentication.
- Errors in security mechanisms, for example, exceeding limits or available attempts.
- Violation of Content Security Policy.
- Errors in cryptographic modules.
- Session management failures, for example, access attempts with expired cookies or an invalid JWT signature on a session token.
- Input validation failures, such as unacceptable encodings, invalid parameter names and values, etc.
- Output validation failures, such as database record set mismatch, invalid data encoding, etc.
- Application errors and system events:
    - Syntax and runtime errors.
    - Connectivity problems.
    - Performance issues.
    - Third-party service error messages.
    - File system errors.
    - File upload virus detection.
    - Configuration changes.
    - etc.
- Use of higher-risk functionality:
    - Network connections.
    - Addition or deletion of users.
    - Changes to privileges.
    - Assigning users to tokens.
    - Adding or deleting tokens.
    - Use of administrative privileges.
    - Access by application administrators.
    - All actions by users with administrative privileges.
    - Access to payment cardholder data.
    - Use of data encrypting keys.
    - Key changes.
    - Creation and deletion of system-level objects.
    - Data import and export including screen-based reports.
    - Submission of user-generated content - especially file uploads.
    - etc.
- Application and related systems start-ups and shut-downs, and logging initialization (starting, stopping or pausing).
- Legal and other opt-ins:
    - Permissions for mobile phone capabilities.
    - Terms of use.
    - Terms & conditions.
    - Personal data usage consent.
    - Permission to receive marketing communication.
    - etc.

# Format of events logged by an application

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

Use the following format to log events in an application:

1. Date and time (use standard time).
1. Component name or ID.
1. Source IP address (optional, if there is an access log).
1. User ID (if available).
1. Event metadata (event ID or type, passed data, etc.)
1. Status (optional, if HTTP status is logged in an access log).
1. Trace ID for "gluing" logs from different places.

# Format of events logged in an access log

<div align="left">
<img src="/.gitbook/assets/type-base-icon.svg">
</div>

Use the following format to log events in an access log:

1. Date and time (use standard time).
1. Component name or ID.
1. HTTP method and path.
1. Source IP address.
1. HTTP status code.
1. HTTP headers that do not contain sensitive data (optional).
1. Request execution time in `ms`.
1. Trace ID for "gluing" logs from different places.

# References

- [OWASP Cheat Sheet Series: Logging](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
