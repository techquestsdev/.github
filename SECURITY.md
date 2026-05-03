# Security Policy

## Reporting a Vulnerability

If you've found a security vulnerability in any Tech Quests project, please
**do not open a public issue**. Instead, report it privately so we can fix it
before it becomes public.

### How to report

- **Preferred:** use GitHub's [private security advisory](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing-information-about-vulnerabilities/privately-reporting-a-security-vulnerability)
  feature on the affected repository (Security tab → "Report a vulnerability").
- **Email:** [contact@techquests.dev](mailto:contact@techquests.dev) — include
  the project name, a description of the issue, reproduction steps, and any
  proof-of-concept if available.

We aim to acknowledge reports within 72 hours and provide an initial assessment
within one week.

## Supported versions

These projects are maintained by a single person on personal time. Security
fixes target the latest released version of each repository unless otherwise
noted in that repository's README.

## Scope

In scope:
- Source code in any repository under the [`techquestsdev` organization](https://github.com/techquestsdev)
- Built artifacts (binaries, container images, Helm charts) published from
  those repositories

Out of scope:
- The `techquests.dev` website itself (it's a static site with no
  user-submitted data) — but please still report obvious issues
- Third-party dependencies — please report those upstream first
