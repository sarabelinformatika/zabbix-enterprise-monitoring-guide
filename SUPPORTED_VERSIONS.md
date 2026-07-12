# Supported Versions

## Documentation Support Policy

The **Zabbix Enterprise Monitoring Guide** is maintained as a continuously evolving documentation project.

Security corrections, technical updates and compatibility clarifications are normally applied only to:

- the latest published release;
- the current development version on the `main` branch.

Older documentation versions may remain available for historical reference but are not actively maintained.

---

## Currently Supported Releases

| Documentation version | Support status |
|---|---|
| `main` / Unreleased | Supported |
| Latest tagged release | Supported |
| Older tagged releases | Not actively supported |

The latest version of the documentation should always be used when planning or modifying production monitoring environments.

---

## Pre-1.0 Development Policy

Until version `1.0.0` is released, the project is considered under active development.

During this period:

- chapter structures may change;
- files may be renamed or reorganized;
- recommendations may be expanded or revised;
- internal links may change;
- incomplete sections may be marked as work in progress.

Pre-1.0 releases should not be treated as permanently stable documentation snapshots.

---

## Zabbix Compatibility

This guide focuses primarily on architectural principles, monitoring strategy and operational best practices rather than release-specific configuration instructions.

Recommendations are intended to remain applicable across currently maintained Zabbix releases where possible.

However, users must verify compatibility with their deployed Zabbix version before implementing:

- agent configurations;
- templates;
- preprocessing rules;
- discovery rules;
- trigger expressions;
- API integrations;
- database settings;
- authentication methods;
- encryption settings;
- proxy configurations.

Features, defaults and supported syntax may differ between Zabbix releases.

---

## Supported Zabbix Releases

The guide is primarily intended for Zabbix versions that are actively supported by the official Zabbix project.

| Zabbix release status | Documentation applicability |
|---|---|
| Current supported LTS release | Primary target |
| Current supported standard release | Generally applicable |
| End-of-life release | Limited applicability |
| Unsupported or modified release | Not supported |

This repository does not provide maintenance or security support for the Zabbix software itself.

For authoritative software lifecycle and support information, consult the official Zabbix documentation.

---

## Operating Systems and Integrations

Examples and recommendations may reference technologies such as:

- Linux;
- Windows Server;
- SNMP-compatible network devices;
- virtualization platforms;
- relational databases;
- reverse proxies;
- SMTP services;
- external authentication systems;
- notification and automation platforms.

Support for a specific operating system, database or integration depends on:

- the deployed Zabbix version;
- the vendor's support lifecycle;
- the configuration of the environment;
- the availability of maintained packages or templates.

Users are responsible for validating third-party compatibility.

---

## End-of-Life Documentation

A documentation version may be considered unsupported when:

- a newer release replaces it;
- the referenced Zabbix release reaches end of life;
- recommendations are no longer operationally safe;
- significant architectural changes make the content obsolete;
- security practices described in the version are outdated.

Unsupported versions may not receive corrections, updates or security-related revisions.

---

## Reporting Compatibility Issues

Compatibility problems may be reported through GitHub Issues.

Reports should include:

- the affected documentation file;
- the Zabbix version;
- the operating system or platform;
- the relevant integration or component;
- a description of the incompatibility;
- supporting documentation where available.

Security-related concerns should be reported according to [SECURITY.md](SECURITY.md).

---

## Recommendation

For production environments:

1. Use the latest supported Zabbix release appropriate for the organization.
2. Consult the latest version of this guide.
3. Verify all commands and settings against official documentation.
4. Test changes in a controlled environment.
5. Maintain backups and rollback procedures.
6. Document all production modifications.
