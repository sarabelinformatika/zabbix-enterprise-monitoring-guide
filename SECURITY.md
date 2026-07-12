# Security Policy

## Purpose

The **Zabbix Enterprise Monitoring Guide** contains documentation only.

This repository does not distribute executable software, deployment packages or production-ready configurations. Security reports should therefore relate to the accuracy, safety and operational impact of the documentation.

---

## Supported Documentation Versions

Security-related corrections are applied to the latest supported release of the guide.

Users should always consult the most recent version before implementing recommendations in production environments.

See [SUPPORTED_VERSIONS.md](SUPPORTED_VERSIONS.md) for the current support policy.

---

## Reporting a Security Issue

Please report concerns if the documentation contains:

- unsafe security recommendations;
- inaccurate authentication guidance;
- insecure monitoring practices;
- recommendations that may expose credentials;
- incorrect access-control guidance;
- misleading network-security information;
- instructions that could negatively affect production environments.

Do not publish sensitive infrastructure information, credentials, private addresses or customer-specific data in public Issues.

---

## Reporting Process

When reporting a potential security issue, include:

- the affected file or chapter;
- a clear description of the concern;
- the potential operational or security impact;
- the recommended correction;
- supporting references where available.

For issues that do not contain sensitive information, a GitHub Issue may be opened.

Sensitive matters should be reported through the contact channels listed on the SARABEL Informatika website.

---

## Scope

Security reports may cover:

- documentation accuracy;
- authentication recommendations;
- access-control practices;
- credential-handling guidance;
- monitoring-system hardening;
- agent and SNMP security;
- data-retention recommendations;
- operational safety;
- infrastructure exposure risks.

The following are outside the scope of this repository:

- vulnerabilities in Zabbix software;
- vulnerabilities in third-party integrations;
- customer-specific infrastructure incidents;
- unsupported or modified software deployments.

Software vulnerabilities should be reported directly to the appropriate vendor or project maintainer.

---

## Responsible Disclosure

Please allow reasonable time for review and correction before publicly disclosing a documentation-related security concern.

The maintainers may:

- request additional information;
- revise the affected documentation;
- publish a corrected release;
- acknowledge valid reports where appropriate.

---

## Operational Safety

Readers remain responsible for validating all recommendations before applying them to production systems.

Security changes should be:

1. Reviewed.
2. Tested in a controlled environment.
3. Documented.
4. Supported by a rollback plan.
5. Monitored after implementation.

---

Thank you for helping maintain the technical accuracy and operational safety of this guide.
