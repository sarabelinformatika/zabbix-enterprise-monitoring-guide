# Agent Monitoring

While SNMP is the preferred monitoring method for most network devices, servers and workstations often require deeper visibility than SNMP can provide. This is the role of the Zabbix Agent.

The agent collects operating system metrics directly from the monitored host and enables detailed performance monitoring, service validation and application-specific checks. Because it operates locally, it has access to information that is either unavailable or impractical to retrieve through external polling.

For most enterprise environments, agent-based monitoring should be considered the primary method for Linux and Windows systems.

---

## Why Use the Zabbix Agent?

A Zabbix Agent can collect a broad range of operational metrics, including:

* CPU utilization and load
* Memory usage
* File system capacity
* Process status
* Operating system services
* Network statistics
* Event logs
* Installed software
* Custom application data

Unlike SNMP, the agent understands the operating system it is running on, making it suitable for monitoring business services as well as infrastructure.

---

## Agent 2

Current enterprise deployments should use **Zabbix Agent 2** unless compatibility requirements dictate otherwise.

Agent 2 provides a modern architecture with improved extensibility and supports numerous plugins for technologies such as:

* PostgreSQL
* MySQL
* Docker
* Microsoft SQL Server
* Redis
* MongoDB
* Kubernetes

Selecting Agent 2 also simplifies future upgrades as new functionality is increasingly delivered through plugins rather than the legacy agent.

---

## Active and Passive Checks

The Zabbix Agent supports two communication models.

### Passive checks

With passive monitoring, the Zabbix Server or Proxy connects to the agent and requests data.

```text
Zabbix Server
      │
      ▼
 Zabbix Agent
```

This approach is simple and easy to troubleshoot, making it suitable for internal networks where direct connectivity exists.

---

### Active checks

With active monitoring, the agent initiates communication with the Zabbix Server or Proxy.

```text
Zabbix Agent
      │
      ▼
Zabbix Server
```

This model is particularly useful for:

* remote offices;
* hosts behind NAT;
* cloud instances;
* environments with restrictive firewalls.

Because the monitored host establishes the connection, fewer inbound firewall rules are required.

---

## Choosing the Right Model

Neither method is universally better.

A common enterprise design is:

| Environment                    | Recommended method |
| ------------------------------ | ------------------ |
| Internal servers               | Passive            |
| Remote offices                 | Active             |
| Cloud workloads                | Active             |
| Workstations                   | Active             |
| High-security internal servers | Passive or Hybrid  |

Many organizations successfully operate a hybrid environment where both methods coexist.

---

## Monitoring Through a Proxy

Remote systems should generally communicate with a local Zabbix Proxy rather than directly with the central server.

```text
Central Zabbix Server
          │
          ▼
     Zabbix Proxy
          │
          ▼
     Linux / Windows Hosts
```

This reduces WAN dependency, allows local buffering during outages and simplifies firewall configuration.

---

## Template Selection

The operating system template forms the foundation of agent monitoring.

Typical examples include:

* Template OS Linux by Zabbix Agent 2
* Template OS Windows by Zabbix Agent 2

Additional templates should be linked only when required by the server's role.

For example, a PostgreSQL database server may inherit:

* Linux Agent 2
* PostgreSQL
* Backup Monitoring

A domain controller or web server should receive a different set of templates.

The monitoring configuration should reflect the purpose of the system rather than the operating system alone.

---

## Custom Checks

One of the greatest strengths of the Zabbix Agent is the ability to collect organization-specific metrics.

Examples include:

* backup age;
* application health;
* license validity;
* synchronization status;
* custom business services.

These checks allow monitoring to move beyond infrastructure and measure actual business availability.

Whenever possible, custom checks should return structured and predictable values that can be reused across multiple hosts.

---

## Security

Because the Zabbix Agent communicates with privileged operating system components, security should be considered from the beginning.

Recommended practices include:

* enabling TLS encryption;
* using PSK or certificates;
* restricting allowed server addresses;
* avoiding unnecessary UserParameters;
* running only trusted scripts;
* reviewing agent logs regularly.

Monitoring systems often become trusted infrastructure. Protect them accordingly.

---

## Performance Considerations

The Zabbix Agent has a very small resource footprint under normal circumstances.

Performance problems usually result from poor monitoring design rather than the agent itself.

Examples include:

* collecting unnecessary metrics;
* executing expensive scripts too frequently;
* monitoring thousands of log entries every few seconds;
* creating duplicate checks.

Before adding a new item, ask whether the collected information will support:

* alerting;
* troubleshooting;
* capacity planning;
* reporting.

If the answer is no, the metric may not be worth collecting.

---

## Operational Best Practices

Enterprise environments benefit from standardized deployment.

A consistent rollout should include:

* identical agent versions where possible;
* standardized configuration;
* reusable templates;
* centralized updates;
* documented custom checks;
* version-controlled configuration.

Avoid configuring every server individually. Consistency greatly reduces long-term operational effort.

---

## Common Mistakes

The following issues are frequently encountered in production environments:

* mixing active and passive checks without a clear design;
* creating custom checks for every host;
* embedding passwords inside scripts;
* granting excessive operating system permissions;
* monitoring metrics that are never reviewed;
* using different agent configurations for identical server roles.

Most of these problems can be avoided through standardization.

---

## Enterprise Checklist

* [ ] Agent 2 deployed
* [ ] Communication model defined
* [ ] TLS configured
* [ ] Standard operating system template linked
* [ ] Custom checks documented
* [ ] Proxy assignment verified
* [ ] Configuration standardized
* [ ] Agent monitored as part of infrastructure health

---

## Summary

The Zabbix Agent provides deep operating system visibility that cannot be achieved through SNMP alone.

A successful enterprise deployment emphasizes consistency over customization. Standardized templates, secure communication and well-designed custom checks create a monitoring platform that scales with the infrastructure instead of becoming increasingly difficult to maintain.

The next chapter introduces web monitoring and explains how Zabbix can validate websites, APIs and business-critical online services from an end-user perspective.

[Next: Web Monitoring →](08-web-monitoring.md)

