# Template Architecture

Templates are one of the most important architectural elements in a Zabbix environment.

They define how monitoring logic is reused across hosts and determine whether the platform remains manageable as the number of monitored systems grows.

A poorly designed template structure creates:

* duplicated configuration;
* inconsistent trigger thresholds;
* difficult upgrades;
* unnecessary database load;
* complex troubleshooting;
* unreliable monitoring coverage.

A well-designed template model provides:

* standardization;
* modularity;
* predictable behavior;
* centralized maintenance;
* scalable host onboarding;
* consistent alerting;
* controlled customization.

Templates should therefore be treated as reusable monitoring components rather than collections of unrelated items.

---

## The Role of Templates

A Zabbix template may contain:

* items;
* dependent items;
* triggers;
* trigger dependencies;
* graphs;
* dashboards;
* low-level discovery rules;
* item prototypes;
* trigger prototypes;
* graph prototypes;
* web scenarios;
* value maps;
* user macros;
* tags.

When a template is linked to a host, its monitoring logic is inherited by that host.

```text
Template
├── Items
├── Triggers
├── Discovery Rules
├── Graphs
├── Macros
└── Tags
        │
        ▼
      Host
```

This inheritance model allows administrators to manage monitoring behavior centrally.

A change made in one template can be applied to every linked host.

---

## Templates as Configuration Code

In a mature monitoring environment, templates should be treated similarly to application configuration or infrastructure code.

They should be:

* versioned;
* documented;
* reviewed;
* tested;
* exported;
* backed up;
* changed through a controlled process.

A template is not merely a frontend object.

It represents executable monitoring logic that can:

* generate load;
* create alerts;
* trigger automation;
* affect service reporting;
* influence operational decisions.

Template changes should therefore be handled with the same discipline as firewall, backup or infrastructure configuration changes.

---

## Template Design Objectives

A good template architecture should satisfy several objectives.

### Reusability

The same template should be usable across multiple hosts with similar roles.

### Predictability

The same monitored condition should produce the same result on equivalent systems.

### Modularity

Templates should be divided into logical functions instead of combining all monitoring into one large object.

### Adaptability

Thresholds and credentials should be configurable without editing template internals.

### Scalability

The number of linked hosts should not make template maintenance impractical.

### Observability

Items and triggers should clearly describe what they monitor and why.

### Safety

Templates should not generate excessive traffic, database writes or false alarms.

---

## Monolithic and Modular Templates

There are two common approaches to template design:

* monolithic templates;
* modular templates.

---

## Monolithic Templates

A monolithic template contains most or all monitoring logic for a host type.

Example:

```text
Template Windows Server Complete
├── Operating system metrics
├── Disk monitoring
├── Service monitoring
├── Event log monitoring
├── SQL monitoring
├── Backup monitoring
├── Certificate monitoring
└── Application checks
```

This may appear convenient because only one template must be linked.

However, it creates several problems:

* unrelated monitoring functions become tightly coupled;
* updates become risky;
* individual components cannot be reused;
* hosts receive unnecessary items;
* troubleshooting becomes more difficult;
* template ownership becomes unclear.

A large template may also contain hundreds or thousands of items that are irrelevant for many hosts.

---

## Modular Templates

A modular architecture separates monitoring by function.

Example:

```text
Template OS Windows
Template App Microsoft SQL Server
Template Service Veeam Backup
Template Certificate Monitoring
Template Hardware Dell iDRAC
```

A database server may then receive:

```text
Host: AACP-BUD-DB-01

Linked templates:
- Template OS Windows
- Template App Microsoft SQL Server
- Template Service Veeam Backup
- Template Certificate Monitoring
```

This design allows the same application template to be reused across multiple operating system or infrastructure roles.

---

## Recommended Template Layers

A practical enterprise template model can be divided into several layers.

```text
Host
├── Operating System Template
├── Hardware Template
├── Application Template
├── Service Template
├── Network Template
└── Organization-Specific Template
```

Each layer should have a clear responsibility.

---

## Operating System Templates

Operating system templates monitor the base system.

Typical metrics include:

* CPU utilization;
* memory utilization;
* file system usage;
* network interfaces;
* processes;
* operating system uptime;
* system load;
* swap usage;
* operating system services;
* event logs;
* package or update state.

Examples:

```text
Template OS Linux by Zabbix Agent 2
Template OS Windows by Zabbix Agent 2
```

Operating system templates should not normally contain application-specific monitoring.

A Windows template should not assume that every Windows host runs:

* Microsoft SQL Server;
* IIS;
* Veeam;
* accounting software;
* domain services.

---

## Hardware Templates

Hardware templates monitor physical infrastructure.

Examples include:

* IPMI;
* iDRAC;
* iLO;
* RAID controllers;
* server sensors;
* storage arrays;
* UPS devices;
* environmental systems.

Typical monitored values include:

* temperature;
* voltage;
* fan status;
* disk state;
* controller state;
* power supply health;
* battery condition;
* chassis status.

Hardware monitoring should remain separate from operating system monitoring because the data source and failure domain are different.

An operating system may appear healthy while the underlying hardware is reporting a predictive disk failure.

---

## Application Templates

Application templates monitor specific software platforms.

Examples:

```text
Template App PostgreSQL
Template App Microsoft SQL Server
Template App Nginx
Template App Apache
Template App Docker
Template App Redis
Template App MySQL
```

Application templates may monitor:

* process availability;
* response time;
* connection count;
* query performance;
* cache utilization;
* transaction rate;
* replication state;
* error counters;
* resource consumption.

Application templates should monitor the application independently of the underlying operating system wherever possible.

---

## Service Templates

Service templates monitor operational functions rather than software products.

Examples:

```text
Template Service Backup Status
Template Service Certificate Expiration
Template Service DNS Resolution
Template Service SMTP Availability
Template Service File Synchronization
```

A service template may be linked to different host types.

For example, certificate monitoring may apply to:

* Linux web servers;
* Windows application servers;
* firewalls;
* reverse proxies;
* load balancers.

This is why service templates should not be embedded unnecessarily into operating system templates.

---

## Network Templates

Network templates monitor devices and communication services.

Examples:

```text
Template Network MikroTik RouterOS
Template Network Cisco Switch
Template Network ICMP Availability
Template Network WAN Link
Template Network SNMP Interface Discovery
```

Network templates commonly use:

* SNMP;
* ICMP;
* simple checks;
* external checks;
* API-based monitoring.

They should account for:

* interface discovery;
* administrative state;
* operational state;
* bandwidth utilization;
* packet loss;
* packet errors;
* device uptime;
* hardware sensors.

---

## Organization-Specific Templates

Some monitoring logic is unique to a business, customer or operational process.

Examples:

```text
Template SARABEL Backup Compliance
Template AACP Application Health
Template Hotel PMS Service Monitoring
Template Customer SLA Monitoring
```

These templates may contain:

* custom scripts;
* business process checks;
* customer-specific thresholds;
* proprietary application checks;
* backup compliance rules;
* local naming standards.

Organization-specific logic should remain separated from vendor templates.

This makes upgrades and troubleshooting easier.

---

## Template Naming Conventions

Template names should indicate their purpose clearly.

A recommended naming pattern is:

```text
Template <Category> <Technology or Function> <Method>
```

Examples:

```text
Template OS Linux Agent 2
Template App PostgreSQL Agent 2
Template Network MikroTik SNMP
Template Service Certificate Expiration
Template Hardware Dell iDRAC
```

Possible categories include:

| Category | Purpose                               |
| -------- | ------------------------------------- |
| OS       | Operating system monitoring           |
| App      | Application monitoring                |
| Network  | Network device or protocol monitoring |
| Service  | Operational service monitoring        |
| Hardware | Physical hardware monitoring          |
| Custom   | Organization-specific monitoring      |

Avoid vague names such as:

```text
Template Server
Template Monitoring
Template New
Template Custom Final
Template Test 2
```

---

## Vendor Templates and Custom Templates

Zabbix provides official templates for many operating systems, applications and devices.

Vendor or official templates offer several advantages:

* maintained item keys;
* standardized triggers;
* discovery rules;
* tested preprocessing;
* documented macros;
* upgrade compatibility.

However, official templates should still be reviewed before production use.

They may:

* collect unnecessary metrics;
* use unsuitable thresholds;
* create excessive item counts;
* assume specific permissions;
* require additional plugins;
* generate alerts that do not match local operations.

Official templates should be treated as a baseline, not an unquestionable final configuration.

---

## Extending Official Templates

Avoid modifying official templates directly where possible.

Direct modification creates upgrade problems because future template imports may overwrite or conflict with local changes.

A better approach is to create a separate custom template.

```text
Template OS Linux by Zabbix Agent
        │
        └── Template Custom Linux Baseline
```

The custom template may add:

* organization-specific triggers;
* custom service checks;
* backup monitoring;
* security controls;
* local macros;
* local tags.

This separates vendor-maintained logic from organization-specific logic.

---

## Template Linking

Templates may be linked directly to hosts or to other templates.

Example template inheritance:

```text
Template Custom Linux Production
├── Template OS Linux Agent 2
├── Template Service Backup
└── Template Service Certificate Monitoring
```

A host may then link only:

```text
Template Custom Linux Production
```

This approach can simplify host onboarding.

However, excessive template inheritance may make troubleshooting difficult.

Administrators should be able to determine where each item or trigger originates.

---

## Template Inheritance Depth

Template inheritance should remain understandable.

A shallow structure is generally preferable:

```text
Host
└── Role Template
    ├── OS Template
    ├── Application Template
    └── Service Template
```

Avoid deeply nested chains:

```text
Host
└── Template A
    └── Template B
        └── Template C
            └── Template D
                └── Template E
```

Deep inheritance creates problems when:

* tracing macros;
* identifying duplicate items;
* reviewing triggers;
* debugging dependencies;
* planning template updates.

---

## Role Templates

A role template combines several functional templates for a specific server role.

Example:

```text
Template Role Windows SQL Server
├── Template OS Windows Agent 2
├── Template App Microsoft SQL Server
├── Template Service Backup
└── Template Service Certificate Monitoring
```

Role templates can simplify provisioning, but they should not duplicate the underlying monitoring logic.

Their purpose is orchestration, not implementation.

---

## Items

Items define the individual values collected by Zabbix.

Each item should have:

* a clear name;
* a unique key;
* an appropriate value type;
* a suitable update interval;
* meaningful units;
* correct preprocessing;
* a defined history policy;
* a defined trend policy.

Example:

```text
Name:
CPU utilization

Key:
system.cpu.util

Type:
Zabbix agent

Value type:
Numeric float

Units:
%
```

---

## Item Naming

Item names should describe the value without requiring the administrator to interpret the key.

Good examples:

```text
CPU utilization
Available memory
File system free space
Backup age
Certificate days remaining
WAN packet loss
```

Poor examples:

```text
CPU
Value 1
Check
Status
Test metric
```

The item name should remain understandable in:

* latest data;
* graphs;
* dashboards;
* problem investigation;
* API output.

---

## Item Keys

Item keys identify the collected metric.

Examples:

```text
system.cpu.util
vm.memory.size[available]
vfs.fs.size[/,pfree]
net.if.in[eth0]
```

Custom item keys should use a structured naming convention.

Example:

```text
sarabel.backup.status
sarabel.backup.age
sarabel.service.health
```

Avoid generic keys such as:

```text
custom.check
script.value
test
```

Unique prefixes reduce the risk of conflicts.

---

## Update Intervals

Update intervals should reflect the operational importance and rate of change of the metric.

Example:

| Metric                 | Recommended interval |
| ---------------------- | -------------------: |
| Critical service state |        30–60 seconds |
| CPU utilization        |           60 seconds |
| Memory utilization     |           60 seconds |
| Disk capacity          |            5 minutes |
| Certificate expiration |           1–12 hours |
| Software inventory     |          12–24 hours |
| Hardware serial number |             24 hours |

Shorter intervals are not automatically better.

Every collected value affects:

* network traffic;
* server processing;
* proxy workload;
* database writes;
* storage growth.

---

## Flexible and Scheduled Intervals

Some checks require different collection frequencies during specific periods.

Examples:

* business-hour monitoring;
* nightly backup validation;
* weekly inventory collection;
* more frequent polling during critical processing windows.

Intervals should be designed around operational needs rather than arbitrary defaults.

---

## History and Trend Retention

Each item may define how long detailed history and trend data are retained.

Detailed history supports:

* incident investigation;
* short-term graphs;
* root cause analysis.

Trend data supports:

* capacity planning;
* long-term reporting;
* historical analysis.

Static or low-value items may not require long history retention.

Retention settings should align with the database capacity plan.

---

## Dependent Items

Dependent items extract multiple values from one master item.

Example:

```text
Master item:
Application API status

Returned data:
{
  "status": "healthy",
  "connections": 42,
  "queue": 3,
  "errors": 0
}
```

Dependent items:

```text
Application health
Active connections
Queue size
Error count
```

This model reduces:

* API requests;
* external script executions;
* device load;
* network traffic;
* duplicated authentication.

Dependent items are strongly recommended for structured data sources such as:

* JSON;
* XML;
* REST APIs;
* command output;
* SNMP walk results.

---

## Preprocessing

Preprocessing transforms collected values before storage.

Common preprocessing methods include:

* JSONPath;
* XML XPath;
* regular expressions;
* JavaScript;
* value mapping;
* trimming;
* numeric conversion;
* rate calculation;
* delta calculation;
* heartbeat validation.

Example:

```text
Raw value:
{"backup_age_seconds": 7200}

JSONPath:
$.backup_age_seconds

Stored value:
7200
```

Preprocessing should remain simple and predictable.

Complex JavaScript processing should be documented and tested carefully.

---

## Preprocessing Failure Handling

A preprocessing failure may cause an item to become unsupported.

Possible strategies include:

* discard the value;
* set a custom error;
* set a default value;
* continue processing.

Failure handling should depend on the metric.

Returning `0` for missing data may create a false impression that a system is healthy.

Unknown and healthy are not equivalent states.

---

## Triggers

Triggers evaluate item values and define problem conditions.

A trigger should answer:

```text
What operational condition requires attention?
```

Examples:

```text
CPU utilization is critically high
Backup has not completed within the expected period
File system free space is below the minimum threshold
Network device is unavailable
Certificate expires within 14 days
```

Triggers should represent actionable conditions.

---

## Trigger Naming

A trigger name should explain:

* the affected component;
* the detected condition;
* the operational significance.

Good examples:

```text
File system free space is below 10%
Latest backup is older than 24 hours
WAN interface packet loss exceeds 20%
PostgreSQL service is not running
```

Poor examples:

```text
Disk problem
Backup error
High value
Warning
Critical issue
```

Problem names should be clear even when displayed outside the host context.

---

## Trigger Expressions

Trigger expressions should be:

* understandable;
* resistant to transient noise;
* based on appropriate time windows;
* configurable with macros;
* documented where complex.

Example:

```text
CPU utilization exceeds 90% for 10 minutes
```

This is usually more useful than:

```text
CPU utilization exceeds 90% once
```

Single-sample conditions often generate unstable alerts.

---

## Threshold Macros

Thresholds should normally be defined using macros.

Example:

```text
{$CPU.UTIL.CRIT}=90
{$DISK.FREE.WARN}=20
{$DISK.FREE.CRIT}=10
{$BACKUP.MAX.AGE}=86400
```

Benefits include:

* per-host overrides;
* customer-specific thresholds;
* easier documentation;
* safer template updates;
* simplified role-based tuning.

Hard-coded thresholds reduce flexibility.

---

## Context Macros

Context macros allow thresholds to vary by discovered entity.

Example:

```text
{$FS.FREE.MIN:"/"}=15
{$FS.FREE.MIN:"/var"}=10
{$FS.FREE.MIN:"/backup"}=5
```

This allows different file systems to use different limits without duplicating triggers.

Context macros are particularly useful for:

* file systems;
* network interfaces;
* services;
* database instances;
* discovered disks.

---

## Trigger Dependencies

Dependencies suppress secondary alarms when a root cause is already known.

Example:

```text
Router unavailable
        │
        ├── Switch unavailable
        ├── Server unavailable
        └── Printer unavailable
```

Without dependencies, one network failure may generate dozens of alerts.

With dependencies:

```text
Primary problem:
Router unavailable

Suppressed secondary symptoms:
Downstream devices unavailable
```

Dependencies should reflect real infrastructure relationships.

Incorrect dependencies may hide genuine problems.

---

## Severity Design

Zabbix trigger severities should have documented operational meanings.

Example:

| Severity       | Recommended interpretation     |
| -------------- | ------------------------------ |
| Not classified | Informational or uncategorized |
| Information    | No immediate action required   |
| Warning        | Review required                |
| Average        | Service degradation            |
| High           | Major operational impact       |
| Disaster       | Critical business interruption |

Severity should reflect business impact, not technical interest.

A failed temperature sensor may be critical on one device and informational on another.

---

## Recovery Expressions

Some conditions require a separate recovery rule.

Example:

```text
Problem:
Packet loss exceeds 20%

Recovery:
Packet loss remains below 5% for 5 minutes
```

Recovery hysteresis prevents repeated problem and recovery transitions around a single threshold.

It is useful for:

* temperature;
* utilization;
* latency;
* packet loss;
* queue length;
* free space.

---

## Event Tags

Triggers should add structured event tags.

Examples:

```text
service=backup
component=database
scope=availability
customer=AACP
priority=critical
```

Event tags support:

* notification routing;
* escalation;
* dashboards;
* service mapping;
* event correlation;
* ticket classification.

Tagging strategy should remain consistent across templates.

---

## Low-Level Discovery

Low-level discovery automatically identifies dynamic resources.

Examples:

* file systems;
* disks;
* network interfaces;
* services;
* containers;
* databases;
* virtual machines;
* certificates.

A discovery rule creates entities from prototypes.

```text
Discovery Rule
├── Item Prototypes
├── Trigger Prototypes
├── Graph Prototypes
└── Host Prototypes
```

Discovery allows one template to adapt to different hosts.

---

## Discovery Filters

Discovery filters determine which entities should be monitored.

Example:

```text
Discover:
- physical interfaces
- active interfaces
- production file systems

Exclude:
- loopback interfaces
- temporary mounts
- virtual adapters
- removable media
```

Unfiltered discovery can create large numbers of unnecessary objects.

---

## Discovery Overrides

Overrides modify discovery behavior for selected entities.

Examples:

* disable monitoring of backup volumes;
* apply different thresholds to system disks;
* suppress triggers on disconnected interfaces;
* increase polling intervals for archival storage;
* assign different severity based on interface role.

Overrides reduce the need to clone templates for minor variations.

---

## Discovery Lifetime

When a discovered entity disappears, Zabbix may:

* disable it;
* retain it;
* remove it after a defined period.

This is important for environments with:

* dynamic containers;
* temporary interfaces;
* removable storage;
* cloud instances;
* changing file systems.

Deleting discovered objects too quickly may remove useful history.

Retaining them indefinitely may create clutter.

---

## Value Maps

Value maps convert numeric or symbolic values into readable states.

Example:

```text
0 → Down
1 → Up
2 → Degraded
3 → Unknown
```

Value maps improve:

* latest data readability;
* dashboards;
* trigger interpretation;
* reporting.

They should be reused consistently across related templates.

---

## Graphs

Graphs provide visual context for collected metrics.

Useful graphs include:

* CPU utilization;
* memory usage;
* file system growth;
* network throughput;
* packet loss;
* application connections;
* database performance;
* backup duration.

Graphs should group related metrics.

Avoid graphs containing too many unrelated values.

A graph should answer a specific operational question.

---

## Template Dashboards

Templates may provide dashboards that become available for linked hosts.

Example dashboard sections:

```text
Host Overview
├── Availability
├── Performance
├── Capacity
├── Application Health
└── Recent Problems
```

Template dashboards improve consistency across similar hosts.

They should remain compact and role-specific.

---

## Web Scenarios

Templates may include web scenarios for application monitoring.

A reusable web template may use macros:

```text
{$WEB.URL}
{$WEB.USER}
{$WEB.PASSWORD}
{$WEB.EXPECTED.TEXT}
```

This allows the same scenario to monitor multiple applications.

Credentials should use secure macros.

---

## Template Macros

Template macros define configurable defaults.

Example:

```text
{$CPU.UTIL.CRIT}=90
{$MEMORY.AVAILABLE.MIN}=10
{$PING.LOSS.MAX}=20
{$CERT.EXPIRE.WARN}=30
```

Macro names should be:

* descriptive;
* consistent;
* documented;
* grouped by function.

Avoid:

```text
{$LIMIT1}
{$VALUE}
{$TEST}
```

---

## Macro Documentation

Every non-obvious macro should include a description.

Example:

```text
Macro:
{$BACKUP.MAX.AGE}

Default:
86400

Description:
Maximum accepted backup age in seconds before a problem is generated.
```

This reduces configuration errors during host onboarding.

---

## Host-Level Overrides

A host may override template defaults.

Example:

```text
Template:
{$DISK.FREE.CRIT}=10

Host override:
{$DISK.FREE.CRIT}=5
```

Overrides should be used only when justified.

Excessive host-level customization makes the environment inconsistent.

When many hosts require the same override, create:

* a role template;
* a customer template;
* a context macro;
* a new default.

---

## Template Tags

Templates should apply standard host or event tags where appropriate.

Examples:

```text
monitoring_source=agent
technology=postgresql
category=database
```

Do not duplicate tags unnecessarily across multiple linked templates.

Conflicting values may complicate filtering and automation.

---

## Custom Scripts

Templates may call custom scripts using:

* user parameters;
* external checks;
* system.run;
* HTTP agents;
* script items;
* Zabbix Sender.

Custom scripts should:

* validate input;
* use strict timeouts;
* return predictable output;
* use meaningful exit codes;
* avoid exposing credentials;
* generate minimal load;
* log failures appropriately.

Scripts should be stored in version control.

---

## User Parameters

User parameters extend Zabbix agent functionality.

Example:

```text
UserParameter=sarabel.backup.status,/usr/local/bin/check-backup.sh
```

The corresponding item key:

```text
sarabel.backup.status
```

User parameters should not execute arbitrary input.

Unsafe parameters may create command injection risks.

---

## Active and Passive Item Strategy

Templates should explicitly define whether items use:

* passive agent checks;
* active agent checks;
* SNMP polling;
* trapper values;
* HTTP agents;
* dependent items.

Avoid mixing monitoring methods without a clear reason.

For remote systems behind NAT, active agent items are often preferable.

For internal servers with direct connectivity, passive checks may be easier to test.

---

## Template Performance

Every template adds workload.

Before deploying a template widely, estimate:

```text
Number of hosts
×
Number of enabled items
÷
Average update interval
```

A template with 200 items linked to 500 hosts creates:

```text
100,000 monitored items
```

At a 60-second average interval:

```text
Approximately 1,667 values per second
```

This does not include:

* discovery;
* preprocessing;
* dependent items;
* trigger evaluation;
* log monitoring;
* event processing.

---

## Avoiding Unnecessary Data Collection

Do not collect data only because it is technically available.

Each item should support:

* alerting;
* troubleshooting;
* capacity planning;
* reporting;
* compliance;
* automation.

Examples of low-value collection:

* static hardware data every minute;
* interface counters for unused adapters;
* detailed process data with no operational purpose;
* hundreds of application metrics never reviewed.

Monitoring should optimize signal, not volume.

---

## Unsupported Items

Templates should be tested for unsupported items before broad deployment.

Common causes include:

* missing agent plugins;
* unsupported operating system versions;
* missing permissions;
* invalid OIDs;
* unavailable APIs;
* incorrect macros;
* preprocessing errors;
* version differences.

Unsupported items should not be ignored.

They may indicate:

* incomplete monitoring;
* incorrect assumptions;
* unnecessary template components;
* deployment failure.

---

## Template Testing

Every new or modified template should be tested on a limited number of hosts.

Recommended test stages:

```text
Development
→ Laboratory
→ Pilot Hosts
→ Limited Production
→ General Deployment
```

Validation should include:

* item support;
* data accuracy;
* trigger behavior;
* recovery behavior;
* discovery output;
* performance impact;
* macro defaults;
* event tags;
* graph readability.

---

## Test Hosts

Maintain one or more dedicated test hosts for template development.

Examples:

```text
LAB-LNX-01
LAB-WIN-01
LAB-SNMP-01
LAB-APP-01
```

Test hosts reduce the risk of generating production alerts during development.

---

## Template Versioning

Templates should include version information.

Example:

```text
Template version: 1.2.0
Maintainer: SARABEL Informatika
Last review: 2026-07-12
Compatible Zabbix version: 7.x
```

Versioning helps track:

* changes;
* compatibility;
* deployment status;
* rollback options.

A semantic model may be used:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
2.1.3
```

---

## Template Change Classification

Changes can be classified by risk.

### Patch change

Examples:

* corrected description;
* improved item name;
* fixed minor preprocessing issue.

### Minor change

Examples:

* added item;
* added trigger;
* added graph;
* introduced new macro.

### Major change

Examples:

* changed item keys;
* changed discovery structure;
* removed monitoring entities;
* changed macro behavior;
* changed trigger semantics.

Major changes require broader testing.

---

## Exporting Templates

Templates should be exported regularly.

Exports provide:

* backup;
* version control;
* migration;
* review;
* rollback;
* documentation.

Common export formats include:

* YAML;
* XML;
* JSON.

YAML is often suitable for human-readable version control.

---

## Version Control Repository

A recommended repository structure:

```text
templates/
├── operating-systems/
├── applications/
├── network/
├── hardware/
├── services/
├── custom/
└── archive/
```

Example:

```text
templates/
├── operating-systems/
│   ├── linux-agent2.yaml
│   └── windows-agent2.yaml
├── network/
│   └── mikrotik-snmp.yaml
└── services/
    └── backup-monitoring.yaml
```

Each template should include supporting documentation.

---

## Template Documentation

Template documentation should describe:

* purpose;
* supported systems;
* required interfaces;
* required macros;
* required permissions;
* item count;
* discovery behavior;
* trigger thresholds;
* known limitations;
* installation requirements;
* test procedure;
* version history.

Example:

```text
Template:
SARABEL Backup Monitoring

Purpose:
Validate backup completion and maximum backup age.

Required macro:
{$BACKUP.MAX.AGE}

Required item source:
Zabbix Sender or custom agent item

Default threshold:
24 hours
```

---

## Template Review Process

A template should be reviewed before production deployment.

Review areas include:

* technical accuracy;
* security;
* naming;
* macro use;
* data volume;
* trigger quality;
* dependencies;
* tag consistency;
* documentation;
* upgrade compatibility.

A second administrator should review high-impact templates where possible.

---

## Template Update Strategy

Template updates should follow a controlled process.

Recommended sequence:

1. export the current template;
2. create a version control commit;
3. test changes in a laboratory environment;
4. link the updated template to pilot hosts;
5. review unsupported items;
6. review generated events;
7. monitor server performance;
8. deploy to production;
9. document the change.

Do not modify widely used templates directly in production without a rollback path.

---

## Unlinking Templates

Removing a template from a host requires careful consideration.

Two common actions are:

* unlink;
* unlink and clear.

### Unlink

Removes the relationship while preserving inherited entities on the host.

### Unlink and clear

Removes the template relationship and inherited configuration.

This may remove:

* items;
* triggers;
* graphs;
* discovery rules;
* web scenarios.

Use `unlink and clear` only when the effect is understood.

---

## Template Migration

Template migration may be required when:

* upgrading Zabbix;
* replacing a custom template;
* adopting an official template;
* changing monitoring methods;
* standardizing multiple environments.

A migration plan should identify:

* item key changes;
* historical data continuity;
* trigger changes;
* macro mapping;
* discovery differences;
* naming conflicts;
* host overrides.

Running old and new templates in parallel temporarily may help validate the transition.

---

## Duplicate Item Keys

Zabbix does not allow duplicate item keys on the same host.

Duplicate keys may occur when:

* two templates monitor the same metric;
* a custom template duplicates an official item;
* template inheritance overlaps;
* a role template contains implementation logic.

Before linking templates, review overlapping items.

---

## Template Conflict Prevention

To reduce conflicts:

* use clear category boundaries;
* use organization-specific key prefixes;
* avoid copying official items unnecessarily;
* document dependencies;
* keep role templates lightweight;
* review inheritance chains;
* test combined template sets.

A template may work correctly alone but fail when linked with others.

---

## Security Considerations

Templates may contain sensitive or high-risk functionality.

Examples include:

* API credentials;
* database passwords;
* SNMP authentication;
* remote commands;
* system.run items;
* custom scripts;
* administrative API access.

Security controls should include:

* secret macros;
* least-privilege accounts;
* restricted scripts;
* encrypted communication;
* code review;
* limited remote command permissions;
* protected exports.

Never commit real credentials into template exports.

---

## Remote Commands

Remote commands may support automation and remediation.

However, they introduce significant risk.

Examples:

* restarting services;
* clearing temporary files;
* triggering scripts;
* restarting agents.

Remote commands should require:

* explicit authorization;
* least-privilege execution;
* logging;
* restricted command lists;
* rollback planning;
* event validation.

Monitoring should not automatically become unrestricted remote administration.

---

## Recommended Template Model

A production Windows application server may use:

```text
Template Role Windows Application Server
├── Template OS Windows Agent 2
├── Template Service ICMP Availability
├── Template Service Backup Compliance
├── Template Service Certificate Expiration
└── Template Custom Application Health
```

A MikroTik router may use:

```text
Template Role MikroTik Edge Router
├── Template Network MikroTik SNMP
├── Template Service ICMP Availability
├── Template Service WAN Monitoring
└── Template Custom Router Security
```

This provides standardization without creating one oversized template.

---

## Template Architecture Example

```text
Templates
├── Operating Systems
│   ├── Linux Agent 2
│   └── Windows Agent 2
├── Applications
│   ├── PostgreSQL
│   ├── Microsoft SQL Server
│   └── Nginx
├── Network
│   ├── MikroTik SNMP
│   ├── Cisco SNMP
│   └── ICMP Availability
├── Services
│   ├── Backup Compliance
│   ├── Certificate Expiration
│   └── DNS Availability
├── Hardware
│   ├── Dell iDRAC
│   ├── HPE iLO
│   └── UPS SNMP
└── Roles
    ├── Linux Web Server
    ├── Windows Application Server
    └── MikroTik Edge Router
```

---

## Template Governance

Template governance defines who may:

* create templates;
* modify templates;
* approve changes;
* deploy templates;
* remove templates;
* manage macros;
* publish exports.

A small organization may use a simple review process.

A larger environment may require:

* development ownership;
* security review;
* operational approval;
* change records;
* version control;
* staged deployment.

---

## Common Template Mistakes

Avoid the following:

* creating one template for every host;
* modifying official templates directly;
* hard-coding passwords;
* hard-coding thresholds;
* collecting unnecessary metrics;
* using very short intervals everywhere;
* creating triggers from single samples;
* failing to define recovery behavior;
* using inconsistent macro names;
* creating deep inheritance chains;
* duplicating item keys;
* deploying untested discovery rules;
* ignoring unsupported items;
* exporting credentials;
* changing production templates without version control.

---

## Template Review Checklist

### Structure

* [ ] Template has a clear purpose
* [ ] Template belongs to a defined category
* [ ] Naming follows the standard
* [ ] Monitoring logic is modular
* [ ] Inheritance is understandable
* [ ] Duplicate functionality has been reviewed

### Items

* [ ] Item names are descriptive
* [ ] Item keys are unique
* [ ] Update intervals are justified
* [ ] Units are configured
* [ ] History retention is appropriate
* [ ] Trend retention is appropriate
* [ ] Preprocessing is tested
* [ ] Unsupported item behavior is understood

### Triggers

* [ ] Trigger names are actionable
* [ ] Thresholds use macros
* [ ] Time windows reduce noise
* [ ] Recovery behavior is defined
* [ ] Dependencies are configured
* [ ] Severity reflects business impact
* [ ] Event tags are standardized

### Discovery

* [ ] Discovery filters are configured
* [ ] Unnecessary entities are excluded
* [ ] Prototypes are reviewed
* [ ] Lost-resource lifetime is defined
* [ ] Overrides are documented
* [ ] Expected item count is estimated

### Security

* [ ] Credentials use secure macros
* [ ] Scripts are reviewed
* [ ] Remote commands are restricted
* [ ] Least-privilege access is used
* [ ] Exports contain no real secrets

### Lifecycle

* [ ] Template version is documented
* [ ] Maintainer is identified
* [ ] Export is stored in version control
* [ ] Test results are documented
* [ ] Rollback path exists
* [ ] Production deployment is approved

---

## Enterprise Recommendations

For a scalable template environment:

* use official templates as validated baselines;
* separate vendor and custom logic;
* create modular templates;
* use role templates only for orchestration;
* define reusable macros;
* use dependent items for structured data;
* control discovery carefully;
* standardize event tags;
* test on pilot hosts;
* store exports in version control;
* document every custom template;
* review performance before mass deployment;
* avoid unnecessary host-level overrides.

---

## Summary

Templates define the reusable monitoring logic of a Zabbix environment.

A strong template architecture provides:

* consistent data collection;
* standardized alerting;
* centralized maintenance;
* predictable onboarding;
* reduced duplication;
* controlled customization;
* scalable operations.

Templates should be modular, documented, versioned and tested.

The next chapter explains how to design and implement reliable SNMP monitoring for network and infrastructure devices.

[Next: SNMP Monitoring →](06-snmp-monitoring.md)

