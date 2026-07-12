# Web Monitoring

Infrastructure may appear healthy while users are unable to access the services that matter most. Servers can be online, databases responsive and network latency low, yet a website or business application may still be unavailable.

Web monitoring bridges this gap by validating services from the user's perspective rather than relying solely on infrastructure metrics.

For many organizations, it represents one of the most valuable layers of monitoring because it measures service availability instead of component availability.

---

## Why Web Monitoring Matters

Traditional infrastructure monitoring answers questions such as:

* Is the server online?
* Is the CPU overloaded?
* Is the database running?

Web monitoring answers different questions:

* Can users access the application?
* Does the login page load?
* Is the API responding?
* Is the SSL certificate valid?
* Is the expected content returned?

These are often the questions that matter most to the business.

---

## Monitoring Beyond Availability

A website returning an HTTP response is not necessarily healthy.

A monitoring strategy should verify that:

* pages load successfully;
* response times remain acceptable;
* expected content is present;
* redirects work correctly;
* authentication succeeds where required;
* APIs return valid responses.

This reduces the risk of false assumptions based solely on server health.

---

## Web Scenarios

Zabbix performs application monitoring through **Web Scenarios**.

A scenario consists of one or more sequential HTTP requests that simulate a user interaction.

For example:

```text id="z0k5q1"
Homepage
      │
      ▼
Login Page
      │
      ▼
Dashboard
```

Each step validates whether the application behaves as expected before continuing to the next request.

---

## What Should Be Monitored?

Not every page requires continuous monitoring.

Priority should be given to services that directly affect business operations.

Typical examples include:

* public websites;
* customer portals;
* APIs;
* authentication services;
* web-based ERP systems;
* monitoring dashboards;
* VPN portals;
* cloud applications.

Monitoring should focus on critical business processes rather than page count.

---

## Response Time

Availability alone provides an incomplete picture.

A website that responds after fifteen seconds is technically available but may still be unusable.

Response time monitoring helps identify:

* overloaded servers;
* slow databases;
* network congestion;
* application bottlenecks;
* external dependency failures.

Thresholds should reflect user expectations rather than arbitrary technical limits.

---

## HTTP Status Codes

HTTP status codes provide valuable operational information.

Typical examples include:

| Code      | Meaning                 |
| --------- | ----------------------- |
| 200       | Successful response     |
| 301 / 302 | Redirect                |
| 401       | Authentication required |
| 403       | Access denied           |
| 404       | Resource not found      |
| 500       | Internal server error   |
| 502       | Bad gateway             |
| 503       | Service unavailable     |

Monitoring should distinguish between expected and unexpected responses.

For example, a login page returning **401 Unauthorized** may be completely normal, while the same response on a public homepage would indicate a problem.

---

## Content Validation

Receiving an HTTP 200 response does not guarantee that the application is functioning correctly.

A maintenance page, application error or reverse proxy message may still return a successful status code.

For this reason, web scenarios should validate expected content whenever possible.

Examples include:

* application title;
* login form;
* company name;
* API response field;
* expected JSON value.

Content validation provides greater confidence that users are seeing the intended application.

---

## SSL Certificate Monitoring

Expired certificates are a common cause of unexpected outages.

Certificate monitoring should verify:

* expiration date;
* validity period;
* certificate chain;
* hostname matching.

Alerts should be generated well before expiration to allow planned renewal rather than emergency replacement.

---

## API Monitoring

Modern applications increasingly expose REST APIs.

These interfaces often support:

* web applications;
* mobile applications;
* third-party integrations;
* automation platforms.

API monitoring should validate:

* availability;
* response time;
* expected status code;
* response structure;
* authentication where applicable.

A healthy API is frequently more important than the public website that depends on it.

---

## Authentication Workflows

Some services require authentication before business functionality becomes available.

Where appropriate, web scenarios can simulate a login sequence.

Typical workflow:

```text id="4x1j8w"
Open Login Page
        │
        ▼
Submit Credentials
        │
        ▼
Verify Dashboard
```

Credentials used for monitoring should have the minimum permissions necessary to complete the validation.

Dedicated monitoring accounts are preferable to administrative accounts.

---

## Geographic Perspective

Users in different locations may experience different levels of service.

A distributed monitoring architecture using multiple Zabbix Proxies allows organizations to validate services from several sites.

```text id="q0hvla"
Budapest Proxy
        │
        ▼
     Web Service
        ▲
        │
Győr Proxy
```

This approach helps distinguish between:

* local connectivity problems;
* WAN failures;
* regional outages;
* application failures.

---

## Internal and External Monitoring

Many organizations operate both internal and public services.

Examples include:

* internal ERP systems;
* VPN portals;
* public websites;
* customer APIs.

Monitoring should occur from locations that reflect actual usage.

An internal application should not necessarily be monitored from the Internet, while a public website should not rely solely on internal monitoring.

---

## Alert Design

Not every failed request requires immediate escalation.

Temporary Internet congestion or short application restarts may produce isolated failures.

Triggers should therefore evaluate sustained conditions rather than individual requests.

Examples include:

* repeated failures;
* increasing response times;
* multiple unsuccessful steps within a scenario.

This reduces alert fatigue while preserving meaningful notifications.

---

## Performance Trends

Historical response times often reveal issues before complete failures occur.

Gradual degradation may indicate:

* increasing database size;
* insufficient server resources;
* storage latency;
* application inefficiencies;
* network congestion.

Trend analysis supports capacity planning and proactive maintenance.

---

## Dependencies

Applications often depend on multiple backend services.

For example:

```text id="g0n6q9"
Website
   │
   ├── Database
   ├── Authentication
   └── API
```

When a shared dependency fails, multiple web scenarios may report problems simultaneously.

Trigger dependencies and service mapping help identify the actual root cause.

---

## Monitoring Third-Party Services

Many organizations rely on external providers.

Examples include:

* Microsoft 365;
* payment gateways;
* cloud APIs;
* identity providers.

Monitoring these services provides visibility into issues that originate outside the organization's own infrastructure.

However, thresholds and escalation policies should account for the limited control available over third-party platforms.

---

## Security Considerations

Web monitoring frequently involves credentials and sensitive endpoints.

Recommended practices include:

* dedicated monitoring accounts;
* secure macro storage;
* TLS validation;
* least-privilege permissions;
* avoiding embedded passwords;
* limiting monitored administrative functions.

Monitoring should observe services, not become an alternative method of accessing them.

---

## Common Mistakes

Typical design problems include:

* monitoring every page instead of critical workflows;
* relying only on HTTP status codes;
* ignoring response time trends;
* using administrator credentials;
* generating alerts after a single failed request;
* monitoring internal services from inappropriate locations.

A small number of well-designed scenarios is generally more valuable than hundreds of superficial checks.

---

## Enterprise Checklist

* [ ] Critical business services identified
* [ ] Web scenarios designed around user workflows
* [ ] Response time monitored
* [ ] Expected content validated
* [ ] SSL certificates monitored
* [ ] APIs included where required
* [ ] Dedicated monitoring accounts used
* [ ] Distributed monitoring considered for remote sites

---

## Summary

Web monitoring measures service availability from the user's perspective rather than focusing solely on infrastructure components.

Well-designed web scenarios validate complete business workflows, helping organizations detect problems that traditional infrastructure monitoring may overlook.

Combined with infrastructure, SNMP and agent monitoring, web monitoring provides a comprehensive view of service health and user experience.

The next chapter explores dashboards and demonstrates how monitoring data can be presented in a clear and actionable way for administrators, management and operational teams.

[Next: Dashboards →](09-dashboards.md)

