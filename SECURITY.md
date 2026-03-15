# Security Policy

## Supported Versions

We actively support and patch the latest stable release of each component in the Power Platform Core organisation. Older versions may not receive security updates.

| Version | Supported |
|---|---|
| Latest release | ✅ Yes |
| Previous minor | ⚠️ Critical fixes only |
| Older versions | ❌ No |

## Reporting a Vulnerability

**Please do NOT report security vulnerabilities through public GitHub issues.**

If you discover a security vulnerability, please report it responsibly by using [GitHub's private security advisory feature](../../security/advisories/new):

1. Navigate to the **Security** tab of the affected repository.
2. Click **"Report a vulnerability"**.
3. Fill in the advisory form with as much detail as possible.

### What to Include

- Description of the vulnerability
- Steps to reproduce the issue
- Potential impact assessment
- Affected component(s) and version(s)
- Any suggested mitigation or fix (optional)

## Response Timeline

| Stage | Target Time |
|---|---|
| Initial acknowledgement | Within 48 hours |
| Severity assessment | Within 5 business days |
| Fix or mitigation | Depends on severity (see below) |
| Public disclosure | After fix is available |

### Severity-Based Fix Timelines

| Severity | Target Fix Time |
|---|---|
| Critical (CVSS ≥ 9.0) | 7 days |
| High (CVSS 7.0–8.9) | 14 days |
| Medium (CVSS 4.0–6.9) | 30 days |
| Low (CVSS < 4.0) | Next scheduled release |

## Disclosure Policy

We follow a **coordinated disclosure** model. We ask that you:

- Allow us reasonable time to investigate and fix the issue before any public disclosure.
- Avoid exploiting the vulnerability beyond what is necessary to demonstrate it.
- Avoid accessing, modifying, or deleting data that does not belong to you.

We will credit you in the release notes and security advisory (unless you prefer to remain anonymous).

## Scope

This policy applies to all repositories under the [`power-platform-core`](https://github.com/power-platform-core) GitHub organisation, including:

- `core` — shared library
- `api-gateway` — API Gateway
- `service-template` — service template
- `infrastructure` — deployment stack

## Out of Scope

The following are **not** considered security vulnerabilities under this policy:

- Vulnerabilities in third-party dependencies (report those upstream)
- Issues that require physical access to a server
- Social engineering attacks
- Denial-of-service attacks that require significant resources to execute
