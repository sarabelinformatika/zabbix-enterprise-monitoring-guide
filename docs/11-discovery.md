
# Discovery

Modern infrastructures change continuously. New virtual machines are deployed, network devices are replaced, cloud instances appear and disappear, and applications evolve over time. Maintaining every monitored object manually quickly becomes impractical.

Discovery allows Zabbix to identify infrastructure automatically and keep the monitoring platform aligned with the actual environment.

When implemented correctly, discovery reduces administrative effort while improving consistency across large deployments.

---

## Why Discovery Matters

Manual host creation works well in small environments, but it does not scale.

As infrastructure grows, manual processes introduce problems such as:

* forgotten hosts;
* inconsistent template assignment;
* missing monitoring;
* outdated inventory;
* duplicate configuration.

Discovery helps ensure that new systems become visible to the monitoring platform as soon as they appear.

---

## Discovery Methods

Zabbix supports several forms of discovery, each designed for a different purpose.

The most common are:

* Network Discovery
* Low-Level Discovery (LLD)
* Active Agent Autoregistration

Although these mechanisms share a common goal, they solve different operational challenges.

---

## Network Discovery

Network Discovery scans a defined network range and identifies reachable devices.

Typical discovery checks include:

* ICMP availability;
* SNMP services;
* Zabbix Agent;
* HTTP;
* HTTPS;
* SSH.

Once a device has been identified, Zabbix can automatically perform predefined actions such as assigning host groups or linking templates.

Network Discovery is particularly useful for:

* branch offices;
* laboratory environments;
* infrastructure audits;
* unmanaged networks.

---

## Low-Level Discovery

Low-Level Discovery (LLD) operates inside an already monitored host.

Rather than discovering hosts, it discovers resources belonging to that host.

Typical examples include:

* file systems;
* network interfaces;
* physical disks;
* Windows services;
* Linux systemd services;
* Docker containers;
* virtual machines.

Instead of creating every monitoring item manually, Zabbix generates them automatically from discovery rules.

This keeps templates reusable across many different systems.

---

## Agent Autoregistration

Active Zabbix Agents can register themselves automatically when they first communicate with the monitoring platform.

A typical workflow is:

```text id="6txg8b"
New Server
      │
      ▼
Active Agent
      │
      ▼
Autoregistration
      │
      ▼
Host Created
      │
      ▼
Templates Assigned
```

This approach is particularly valuable in environments where servers are deployed automatically through virtualization or cloud platforms.

---

## Discovery Actions

Discovery itself only identifies new objects.

Operational value comes from **actions** that are executed after discovery.

Typical actions include:

* creating a host;
* assigning host groups;
* linking templates;
* assigning tags;
* selecting a proxy;
* enabling monitoring.

By automating these steps, organizations ensure that every new system follows the same operational standards.

---

## Standardization

Discovery should reinforce standardization rather than replace it.

Every automatically created host should receive:

* consistent naming;
* appropriate templates;
* standard tags;
* predefined monitoring policies.

Automation should never create partially configured hosts that require extensive manual correction.

---

## Filtering

Not every discovered object should be monitored.

Discovery rules should include appropriate filters.

Examples include:

* excluding temporary file systems;
* ignoring inactive interfaces;
* skipping removable storage;
* excluding virtual adapters;
* limiting discovery to production networks.

Good filtering improves both monitoring quality and platform performance.

---

## Discovery and Templates

Discovery works best when combined with modular templates.

For example, a Linux template may automatically discover:

* mounted file systems;
* network interfaces;
* system services.

The template remains identical across all Linux servers while adapting dynamically to the resources available on each system.

This eliminates the need to create different templates for every server configuration.

---

## Discovery in Enterprise Environments

Large environments often combine multiple discovery methods.

For example:

```text id="7rjyzl"
Network Discovery
        │
        ▼
Host Created
        │
        ▼
Agent Autoregistration
        │
        ▼
Low-Level Discovery
```

Each layer builds upon the previous one, creating an automated onboarding process with minimal manual intervention.

---

## Security Considerations

Discovery mechanisms should be carefully controlled.

Poorly configured discovery can:

* create unwanted hosts;
* monitor unauthorized systems;
* generate unnecessary traffic;
* expose management interfaces.

Best practices include:

* limiting discovery ranges;
* using dedicated credentials;
* assigning least-privilege permissions;
* reviewing discovery actions regularly.

Automation should always remain predictable.

---

## Performance

Aggressive discovery schedules can increase monitoring load without providing additional value.

Infrastructure rarely changes every few minutes.

Discovery intervals should therefore reflect the expected rate of change.

Examples include:

| Environment               | Suggested Interval |
| ------------------------- | ------------------ |
| Stable production network | Daily              |
| Virtualization cluster    | Every few hours    |
| Cloud environment         | Hourly             |
| Laboratory                | As required        |

Choosing an appropriate interval reduces unnecessary scanning while keeping inventory current.

---

## Common Mistakes

Frequently encountered issues include:

* scanning networks that should not be monitored;
* discovering every available interface;
* creating duplicate hosts;
* assigning incorrect templates;
* failing to review discovery actions;
* relying entirely on automatic onboarding.

Automation should simplify administration, not eliminate operational oversight.

---

## Enterprise Recommendation

Discovery should be viewed as an onboarding mechanism rather than a replacement for asset management.

In mature environments, newly discovered systems should become part of a documented lifecycle that includes ownership, security validation, template assignment and inventory verification before being considered production-ready.

Automation accelerates deployment, but governance ensures consistency.

---

## Enterprise Checklist

* [ ] Discovery method selected
* [ ] Discovery ranges documented
* [ ] Standard templates assigned automatically
* [ ] Host groups and tags applied consistently
* [ ] Discovery filters reviewed
* [ ] Discovery intervals optimized
* [ ] Security restrictions implemented
* [ ] Automatic onboarding validated

---

## Summary

Discovery enables Zabbix to adapt automatically as infrastructure evolves.

By combining Network Discovery, Low-Level Discovery and Agent Autoregistration, organizations can significantly reduce administrative effort while maintaining consistent monitoring standards.

Automation is most effective when supported by well-designed templates, standardized policies and controlled operational processes.

The next chapter explores security and explains how to protect the monitoring platform, monitored systems and sensitive operational data.

[Next: Security →](12-security.md)
