# Security Policy

We take security and user privacy seriously.

## Reporting a Vulnerability

- Email: **security@example.com**
- GPG: (optional) Provide your key fingerprint and a public key link.
- Please include a minimal reproduction and potential impact.

We aim to acknowledge within **48 hours** and provide a timeline for remediation.  
Do not disclose vulnerabilities publicly until we release a fix and coordinated advisory.

## Scope

- Core server and web UI
- Plugin host and official connectors
- SDKs (`plugin-sdk-js`, `plugin-sdk-py`)

## Out of Scope

- Third-party plugins maintained by community members (report to their maintainers first).

## Handling Sensitive Data

- Never log secrets or PII.
- Redact PII (names, emails, addresses, phone numbers) in traces and logs.
- Presidio redaction is applied to resumes before storage.

## Supported Versions

Pre-1.0: latest `main` and the most recent minor release.  
Post-1.0: last **two** minor releases receive security fixes.
