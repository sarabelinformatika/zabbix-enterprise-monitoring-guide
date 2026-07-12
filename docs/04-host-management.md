# Host Management

Hosts are the central objects of a Zabbix monitoring environment.

A host represents any monitored entity, including:

* physical servers;
* virtual machines;
* routers;
* switches;
* firewalls;
* storage systems;
* hypervisors;
* applications;
* websites;
* cloud services;
* environmental sensors;
* UPS devices;
* printers;
* containers.

Effective host management requires more than creating individual objects.

A scalable design must also define:

* naming conventions;
* host groups;
* interfaces;
* templates;
* macros;
* inventory;
* tags;
* proxy assignments;
* encryption settings;
* status and lifecycle rules.

---

## Host Management Overview

Each host contains the configuration required for Zabbix to collect and interpret monitoring data.

A typical host configuration includes:

```text
Host
├── Host name
├── Visible name
├── Host groups
├── Interfaces
├── Templates
├── Tags
├── Macros
├── Inventory
├── Proxy assignment
├── Encryption settings
└── Monitoring status
```

The design should remain consistent across all environments.

Inconsistent host configuration leads to:

* duplicate hosts;
* missing data;
* incorrect trigger behavior;
* difficult filtering;
* permission problems;
* broken active checks;
* maintenance errors;
* poor reporting.

---

## Host Naming Conventions

The host name is the primary technical identifier used by Zabbix.

It should be:

* unique;
* stable;
* predictable;
* machine-readable;
* independent of temporary descriptions;
* consistent with agent configuration.

A recommended naming format is:

```text
CUSTOMER-SITE-ROLE-NUMBER
```

Examples:

```text
SAR-BUD-ZBX-01
SAR-BUD-PVE-01
SAR-BUD-FW-01
AACP-BUD-SRV-01
AACP-BUD-SW-01
HOTEL-GYR-AP-03
```

Possible naming components:

| Component | Example | Meaning                  |
| --------- | ------- | ------------------------ |
| Customer  | AACP    | Customer or organization |
| Site      | BUD     | Physical location        |
| Role      | SRV     | Server role              |
| Number    | 01      | Sequence number          |

---

## Recommended Role Codes

Example role abbreviations:

| Device type                  | Code |
| ---------------------------- | ---- |
| Zabbix Server                | ZBX  |
| Zabbix Proxy                 | ZXP  |
| Physical Server              | SRV  |
| Virtual Machine              | VM   |
| Domain Controller            | DC   |
| File Server                  | FS   |
| Terminal Server              | TS   |
| Hypervisor                   | HV   |
| Proxmox Host                 | PVE  |
| Router                       | RTR  |
| Firewall                     | FW   |
| Switch                       | SW   |
| Wireless Access Point        | AP   |
| Network Attached Storage     | NAS  |
| Uninterruptible Power Supply | UPS  |
| Printer                      | PRN  |
| Database Server              | DB   |
| Web Server                   | WEB  |
| Application Server           | APP  |

The abbreviations should be documented and reused consistently.

---

## Host Name and Visible Name

Zabbix distinguishes between:

* host name;
* visible name.

### Host name

The host name is the internal technical identifier.

Example:

```text
AACP-BUD-SRV-01
```

It is used by:

* active agent checks;
* API operations;
* template logic;
* automation;
* discovery;
* integrations.

### Visible name

The visible name is displayed in the frontend.

Example:

```text
AACP Primary Application Server
```

The visible name may be more descriptive than the host name.

Recommended configuration:

```text
Host name: AACP-BUD-SRV-01
Visible name: AACP Primary Application Server
```

The host name should remain stable even if the server function changes slightly.

---

## Avoid Ambiguous Names

Avoid host names such as:

```text
Server
Server1
NewServer
MikroTik
MainPC
NAS
Switch
Test
Temp
```

These names do not identify:

* the customer;
* the location;
* the role;
* the device sequence;
* the environment.

Ambiguous names create operational problems when multiple sites or customers are monitored.

---

## Create a Host

Navigate to:

```text
Data collection
→ Hosts
→ Create host
```

Configure the basic fields.

Example:

```text
Host name: AACP-BUD-SRV-01
Visible name: AACP Application Server
```

Assign at least one host group.

Example:

```text
Customers/AACP/Servers
Operating Systems/Windows
Sites/Budapest
```

Add the appropriate interface and templates.

Save the host.

---

## Host Groups

Host groups organize hosts according to operational, technical or administrative boundaries.

Host groups are used for:

* access control;
* filtering;
* dashboards;
* maintenance periods;
* actions;
* reporting;
* mass updates;
* problem views;
* API automation.

A host may belong to multiple groups.

---

## Recommended Host Group Structure

A hierarchical naming model improves clarity.

Example:

```text
Customers
├── AACP
│   ├── Servers
│   ├── Network
│   ├── Workstations
│   ├── Applications
│   └── Infrastructure
├── Customer-B
│   ├── Servers
│   └── Network
└── Customer-C
    ├── Servers
    └── Network
```

Additional technical groups:

```text
Operating Systems
├── Linux
├── Windows
└── Network Devices

Infrastructure
├── Zabbix Servers
├── Zabbix Proxies
├── Hypervisors
├── Storage
└── Backup Systems

Sites
├── Budapest
├── Győr
├── Debrecen
└── Remote
```

---

## Host Group Design Principles

A host group should represent a useful operational boundary.

Good examples:

```text
Customers/AACP/Servers
Customers/AACP/Network
Sites/Budapest
Operating Systems/Linux
Infrastructure/Proxmox
```

Poor examples:

```text
Important
New
Old
Temporary
Other
Miscellaneous
```

Avoid creating groups without a clear long-term purpose.

---

## Host Interfaces

Interfaces define how Zabbix reaches a monitored host.

Supported interface types include:

* Agent;
* SNMP;
* JMX;
* IPMI.

A host can have multiple interfaces.

Example:

```text
Agent interface:
192.168.10.20
TCP 10050

SNMP interface:
192.168.10.1
UDP 161
```

---

## Agent Interface

The agent interface is used for passive Zabbix agent checks.

Example:

```text
Type: Agent
Connect to: IP
IP address: 192.168.10.20
Port: 10050
Default: Yes
```

The server or proxy must be able to reach the configured address.

Test connectivity:

```bash
nc -zv 192.168.10.20 10050
```

Using Zabbix Get:

```bash
zabbix_get -s 192.168.10.20 -k agent.ping
```

Expected output:

```text
1
```

---

## IP Address or DNS Name

A host interface may use:

* IP address;
* DNS name.

### Use IP address when

* the address is static;
* DNS is unreliable;
* the device is a network appliance;
* monitoring must continue during DNS failure.

### Use DNS when

* addresses may change;
* reliable internal DNS exists;
* the environment uses dynamic addressing;
* cloud instances are monitored by name.

Example DNS interface:

```text
Connect to: DNS
DNS name: server01.example.local
Port: 10050
```

DNS resolution must work from the Zabbix server or assigned proxy.

Test:

```bash
getent hosts server01.example.local
```

---

## SNMP Interface

The SNMP interface is used for SNMP polling.

Example:

```text
Type: SNMP
IP address: 192.168.10.1
Port: 161
Default: Yes
```

Test SNMP connectivity:

```bash
snmpwalk -v2c \
    -c COMMUNITY_STRING \
    192.168.10.1 \
    1.3.6.1.2.1.1
```

For SNMPv3:

```bash
snmpwalk -v3 \
    -l authPriv \
    -u zabbix \
    -a SHA \
    -A AUTH_PASSWORD \
    -x AES \
    -X PRIVACY_PASSWORD \
    192.168.10.1 \
    1.3.6.1.2.1.1
```

SNMPv3 is recommended where supported.

---

## Multiple Interfaces

A host may require more than one interface.

Example MikroTik router:

```text
SNMP interface:
192.168.10.1
UDP 161

Agent interface:
Not configured
```

Example Linux server with IPMI:

```text
Agent interface:
192.168.10.20
TCP 10050

IPMI interface:
192.168.20.20
UDP 623
```

Example Java application server:

```text
Agent interface:
192.168.10.30
TCP 10050

JMX interface:
192.168.10.30
TCP 12345
```

Each interface should exist only if it is actively used.

---

## Default Interface

When multiple interfaces of the same type exist, one must be marked as default.

Example:

```text
Agent interface 1:
192.168.10.20
Default: Yes

Agent interface 2:
10.10.10.20
Default: No
```

Items without an explicitly selected interface use the default interface.

Incorrect default selection can cause monitoring failures.

---

## Host Status

A Zabbix host may be:

* enabled;
* disabled.

### Enabled host

Zabbix collects data and evaluates triggers.

### Disabled host

Zabbix does not poll the host or process new item values.

Use disabled status when:

* the device is not yet deployed;
* configuration is incomplete;
* the host is temporarily removed;
* troubleshooting requires complete monitoring suspension;
* the device is retained for documentation before deletion.

Do not disable hosts merely to suppress expected alarms.

Use maintenance mode instead.

---

## Templates

Templates contain reusable monitoring configuration.

A host can inherit:

* items;
* triggers;
* graphs;
* dashboards;
* discovery rules;
* value maps;
* web scenarios;
* tags;
* macros.

Example Windows server templates:

```text
Windows by Zabbix agent
Microsoft SQL Server
Website certificate monitoring
Custom backup monitoring
```

Example MikroTik router templates:

```text
MikroTik RouterOS by SNMP
ICMP Ping
Custom WAN Monitoring
```

Templates are covered in detail in the next chapter.

---

## Linking Templates

Navigate to the host configuration and select:

```text
Templates
→ Link new templates
```

Search for the required template.

Example:

```text
Linux by Zabbix agent
```

Add the template and save the host.

Inherited entities become visible in the host configuration.

---

## Template Inheritance

A host inherits monitoring entities from linked templates.

```text
Template
├── Items
├── Triggers
├── Graphs
└── Discovery Rules
        │
        ▼
      Host
```

Changes made to the template are applied to all linked hosts.

This reduces configuration duplication.

---

## Unlink and Clear

Zabbix provides two template removal options:

* unlink;
* unlink and clear.

### Unlink

Removes the template relationship but leaves inherited entities on the host.

### Unlink and clear

Removes the relationship and deletes the inherited entities.

Use `Unlink and clear` carefully.

It may remove:

* collected item definitions;
* triggers;
* graphs;
* discovery rules;
* associated configuration.

Historical data behavior depends on the removed entities and database retention.

---

## Host Macros

Macros store reusable configuration values.

Host-level macros apply only to a specific host.

Example:

```text
{$SNMP_COMMUNITY}
{$AGENT.TIMEOUT}
{$BACKUP.MAX.AGE}
{$CPU.UTIL.MAX}
```

A host macro can override a template macro.

Example template macro:

```text
{$CPU.UTIL.MAX}=90
```

Host-specific override:

```text
{$CPU.UTIL.MAX}=95
```

This allows one host to use a different threshold without modifying the template.

---

## Secret Macros

Sensitive values should use protected macro types.

Examples:

* secret text;
* vault-backed secrets where supported;
* protected credentials;
* SNMP authentication values;
* API tokens;
* database passwords.

Do not store secrets in:

* host names;
* visible names;
* item names;
* descriptions;
* scripts committed to repositories;
* unprotected user macros.

Example secret macros:

```text
{$SNMP.USER}
{$SNMP.AUTH.PASSWORD}
{$SNMP.PRIV.PASSWORD}
{$API.TOKEN}
```

---

## Macro Precedence

Macros may be defined at multiple levels:

```text
Global macro
    │
    ▼
Template macro
    │
    ▼
Host macro
```

More specific definitions override broader definitions.

A host-level macro normally overrides the same macro inherited from a template.

Example:

```text
Global:
{$PING.LOSS.MAX}=20

Template:
{$PING.LOSS.MAX}=10

Host:
{$PING.LOSS.MAX}=5
```

The host uses:

```text
5
```

Macro behavior should be documented to avoid unexpected trigger thresholds.

---

## Host Tags

Tags attach structured metadata to hosts, items, triggers and events.

Examples:

```text
customer=AACP
site=Budapest
environment=production
service=accounting
priority=high
owner=SARABEL
```

Tags are useful for:

* problem filtering;
* action conditions;
* event correlation;
* dashboards;
* maintenance;
* notification routing;
* service-level reporting.

---

## Recommended Tag Model

Use a small, consistent tag vocabulary.

Example:

| Tag              | Example value |
| ---------------- | ------------- |
| customer         | AACP          |
| site             | Budapest      |
| environment      | production    |
| device_type      | server        |
| operating_system | Windows       |
| service          | file-server   |
| priority         | critical      |
| owner            | SARABEL       |

Avoid inconsistent variants:

```text
site=Budapest
location=BUD
office=Budapest
city=BP
```

Use one standard tag name.

---

## Tag Naming Rules

Recommended rules:

* use lowercase tag names;
* avoid spaces;
* use predictable values;
* avoid duplicate meaning;
* document allowed values;
* do not store secrets;
* do not use tags as free-form notes.

Good examples:

```text
environment=production
customer=AACP
priority=critical
```

Poor examples:

```text
Environment=Prod
env=Live
priority=Very Important Server
notes=Password is admin123
```

---

## Inventory

Host inventory stores descriptive asset information.

Typical inventory fields include:

* asset tag;
* serial number;
* model;
* vendor;
* operating system;
* hardware;
* software;
* physical location;
* contact person;
* installation date;
* warranty expiration;
* MAC address;
* IP address;
* site notes.

Inventory modes include:

* disabled;
* manual;
* automatic.

---

## Manual Inventory

In manual mode, administrators enter values directly.

Example:

```text
Type: Virtual Server
Name: AACP Application Server
OS: Windows Server
Serial number: VM-AACP-001
Location: Budapest
Vendor: Microsoft
```

Manual inventory is useful for fields that cannot be collected automatically.

---

## Automatic Inventory

Automatic inventory values are populated by monitored items.

Example item configuration:

```text
Item:
system.uname

Inventory field:
OS
```

Another example:

```text
Item:
system.hw.serialnumber

Inventory field:
Serial number
```

Automatic inventory reduces manual administration.

However, it depends on:

* supported item keys;
* permissions;
* operating system capabilities;
* template configuration;
* reliable data collection.

---

## Recommended Inventory Fields

At minimum, consider maintaining:

```text
Name
Type
Model
Serial number
Asset tag
Operating system
Location
Vendor
Contact
Installation date
Hardware
Software
```

For network devices:

```text
Device type
Model
Serial number
Firmware version
Management IP
Physical location
Rack position
Support expiration
```

---

## Proxy Assignment

A host may be monitored:

* directly by the Zabbix server;
* through a Zabbix proxy;
* through a proxy group where supported.

Example direct monitoring:

```text
Monitored by: Server
```

Example proxy monitoring:

```text
Monitored by: Proxy
Proxy: AACP-BUD-ZXP-01
```

The selected proxy performs:

* agent polling;
* SNMP polling;
* ICMP checks;
* external checks;
* web checks;
* discovery;
* local buffering.

---

## When to Assign a Proxy

Use a proxy when:

* the host is at a remote site;
* WAN connectivity is unstable;
* the monitored network is behind NAT;
* firewall access must be restricted;
* customer environments must be isolated;
* local buffering is required;
* polling latency is high.

Example:

```text
Central Zabbix Server
        │
        ▼
AACP-BUD-ZXP-01
        │
        ▼
AACP-BUD-SRV-01
```

---

## Proxy Assignment Consistency

Hosts at the same remote site should normally use the same local proxy.

Avoid configurations where:

```text
Remote server → Proxy
Remote switch → Central server
Remote router → Different proxy
```

unless there is a documented technical reason.

Inconsistent proxy assignment complicates:

* routing;
* firewall rules;
* troubleshooting;
* data collection;
* outage analysis.

---

## Agent Passive Check Configuration

For passive checks, configure the monitored agent.

Linux Agent 2 example:

```text
Server=192.168.10.20
Hostname=AACP-BUD-SRV-01
```

Windows Agent 2 example:

```text
Server=192.168.10.20
Hostname=AACP-BUD-SRV-01
```

When using a proxy:

```text
Server=192.168.20.10
Hostname=AACP-BUD-SRV-01
```

The `Server` parameter should contain the IP address of the Zabbix server or proxy allowed to query the agent.

---

## Agent Active Check Configuration

For active checks:

```text
ServerActive=192.168.10.20
Hostname=AACP-BUD-SRV-01
```

When using a proxy:

```text
ServerActive=192.168.20.10
Hostname=AACP-BUD-SRV-01
```

The hostname must match the Zabbix frontend exactly.

Example mismatch:

```text
Agent:
Hostname=AACP-SRV-01

Frontend:
AACP-BUD-SRV-01
```

This causes active checks to fail.

---

## Host Encryption

Zabbix agent and proxy communication may use:

* no encryption;
* pre-shared key encryption;
* certificate-based encryption.

Host encryption settings define:

* accepted incoming connections;
* outbound connection type;
* PSK identity;
* PSK value;
* certificate issuer;
* certificate subject.

---

## Pre-Shared Key Encryption

A PSK-based host configuration includes:

```text
Connections to host:
PSK

Connections from host:
PSK

PSK identity:
AACP-BUD-SRV-01

PSK:
Generated secret value
```

Generate a PSK on Linux:

```bash
openssl rand -hex 32
```

Example output:

```text
a38f54f77f82c0183df4f7311234567890abcdef1234567890abcdef12345678
```

Store it securely:

```bash
mkdir -p /etc/zabbix/keys
chmod 700 /etc/zabbix/keys
```

Create the key file:

```bash
nano /etc/zabbix/keys/agent.psk
```

Set permissions:

```bash
chown zabbix:zabbix /etc/zabbix/keys/agent.psk
chmod 600 /etc/zabbix/keys/agent.psk
```

---

## Agent PSK Configuration

Example Agent 2 configuration:

```text
TLSConnect=psk
TLSAccept=psk
TLSPSKIdentity=AACP-BUD-SRV-01
TLSPSKFile=/etc/zabbix/keys/agent.psk
```

Restart the agent:

```bash
systemctl restart zabbix-agent2
```

Review logs:

```bash
journalctl -u zabbix-agent2 -n 100 --no-pager
```

The PSK identity and value must match the frontend host configuration.

---

## Host Description

The description field should contain operationally useful information.

Example:

```text
Primary application server for AACP.

Services:
- RLB application
- PostgreSQL database
- File synchronization

Owner:
SARABEL Informatika

Backup:
Daily incremental backup
Weekly full backup

Maintenance window:
Sunday 20:00–22:00
```

Avoid placing sensitive credentials in descriptions.

---

## Host-Level Monitoring Configuration

Host-specific configuration should be used only when necessary.

Possible host-level entities include:

* custom items;
* custom triggers;
* custom graphs;
* host macros;
* host tags;
* inventory fields.

Where configuration is reusable, create a template instead.

Example:

Bad approach:

```text
Create the same backup item manually on 30 hosts
```

Better approach:

```text
Create one backup monitoring template
Link it to 30 hosts
```

---

## Clone a Host

Cloning is useful when creating similar devices.

Navigate to:

```text
Data collection
→ Hosts
→ Select host
→ Clone
```

Review all cloned settings:

* host name;
* visible name;
* interfaces;
* macros;
* inventory;
* tags;
* proxy;
* encryption;
* templates.

Do not save a clone without changing unique identifiers.

Common cloning errors include:

* duplicate IP addresses;
* duplicate host names;
* copied PSK identities;
* incorrect proxy assignments;
* inherited customer tags;
* copied serial numbers.

---

## Mass Update

Mass update modifies multiple hosts simultaneously.

Navigate to:

```text
Data collection
→ Hosts
→ Select hosts
→ Mass update
```

Possible changes include:

* host groups;
* templates;
* macros;
* tags;
* proxy assignment;
* status;
* inventory mode;
* encryption settings.

Mass updates should be used carefully.

A configuration error may affect hundreds of hosts.

Before applying:

1. filter the host list;
2. review selected hosts;
3. modify only required fields;
4. verify the result on a small test group where possible.

---

## Host Import

Hosts can be imported from configuration files.

Supported import formats may include:

* YAML;
* XML;
* JSON.

Navigate to:

```text
Data collection
→ Hosts
→ Import
```

Import options control whether Zabbix should:

* create new objects;
* update existing objects;
* delete missing objects.

Review import rules before execution.

Incorrect deletion options may remove valid configuration.

---

## Host Export

Hosts can be exported for:

* backup;
* documentation;
* migration;
* version control;
* test environment deployment.

Navigate to:

```text
Data collection
→ Hosts
→ Select hosts
→ Export
```

Exported configuration may include:

* host definitions;
* interfaces;
* templates;
* macros;
* groups;
* inventory settings.

Sensitive information should be reviewed before committing exports to a repository.

---

## API-Based Host Management

The Zabbix API can automate host creation and updates.

Typical automation tasks include:

* creating hosts;
* assigning groups;
* linking templates;
* configuring interfaces;
* assigning proxies;
* setting macros;
* updating inventory;
* disabling retired hosts.

Example workflow:

```text
Asset Management System
        │
        ▼
Automation Script
        │
        ▼
Zabbix API
        │
        ▼
Host Created and Configured
```

API automation should enforce the same naming and tagging standards as manual administration.

---

## Host Discovery

Hosts may be created or identified through:

* network discovery;
* active agent autoregistration;
* low-level discovery;
* external automation;
* cloud integrations.

Discovered hosts should not remain unmanaged.

An automated workflow should assign:

* host groups;
* templates;
* tags;
* proxy;
* monitoring method;
* ownership.

Discovery is covered in a later chapter.

---

## Host Autoregistration

Active agents can autoregister with Zabbix.

Typical autoregistration metadata:

```text
Host name
IP address
Agent metadata
Proxy
```

Example agent configuration:

```text
ServerActive=192.168.10.20
Hostname=AACP-BUD-SRV-01
HostMetadata=windows-server
```

An autoregistration action can:

* add the host;
* assign groups;
* link templates;
* enable monitoring;
* add tags.

Autoregistration is useful for:

* automated server deployment;
* cloud instances;
* large workstation fleets;
* standardized virtual machines.

---

## Host Lifecycle Management

Every host should follow a defined lifecycle.

Example:

```text
Planned
→ Deployed
→ Monitoring Enabled
→ Production
→ Maintenance
→ Decommissioned
→ Disabled
→ Removed
```

Lifecycle procedures should define:

* who creates hosts;
* who approves monitoring;
* when hosts are disabled;
* how long decommissioned hosts remain;
* when configuration is exported;
* when historical data may be removed.

---

## Onboarding a New Host

Recommended onboarding process:

1. verify ownership;
2. define host name;
3. identify customer and site;
4. determine monitoring method;
5. select proxy;
6. configure agent or SNMP;
7. create the host;
8. assign groups;
9. link templates;
10. configure macros;
11. add tags;
12. configure inventory;
13. enable encryption;
14. test data collection;
15. validate triggers;
16. document the host.

---

## Host Onboarding Checklist

### Identification

* [ ] Customer identified
* [ ] Site identified
* [ ] Device role identified
* [ ] Host name follows naming standard
* [ ] Visible name is descriptive
* [ ] IP address or DNS name verified

### Monitoring configuration

* [ ] Correct interface configured
* [ ] Correct proxy assigned
* [ ] Required templates linked
* [ ] Host macros configured
* [ ] Tags configured
* [ ] Inventory mode selected
* [ ] Encryption configured

### Validation

* [ ] Host reachable
* [ ] Agent or SNMP test successful
* [ ] Latest data received
* [ ] Unsupported items reviewed
* [ ] Trigger state reviewed
* [ ] Dashboards display expected data
* [ ] Documentation completed

---

## Decommissioning a Host

Do not delete a host immediately when a device is retired.

Recommended process:

1. confirm the device is permanently decommissioned;
2. disable the host;
3. add a decommissioned tag;
4. record the retirement date;
5. export host configuration;
6. preserve required inventory data;
7. confirm retention requirements;
8. remove the host after the defined retention period.

Example tag:

```text
lifecycle=decommissioned
```

Example description:

```text
Decommissioned: 2026-07-12
Replacement: AACP-BUD-SRV-02
Approved by: Infrastructure Manager
```

---

## Disabled Host Retention

A disabled host may be retained temporarily for:

* historical reference;
* audit requirements;
* replacement validation;
* documentation;
* incident investigation.

Define a retention policy.

Example:

```text
Disabled hosts retained for 90 days.
Configuration exported before deletion.
```

Without a policy, obsolete hosts accumulate and reduce administrative clarity.

---

## Duplicate Host Detection

Duplicate hosts may occur because of:

* inconsistent naming;
* manual creation;
* autoregistration;
* network discovery;
* cloned virtual machines;
* reused IP addresses.

Indicators include:

* two hosts with the same IP;
* repeated active check errors;
* identical serial numbers;
* duplicate visible names;
* data appearing under the wrong host.

Review duplicates regularly.

---

## Cloned Virtual Machines

Cloned virtual machines require special attention.

Before starting the clone:

* change the operating system hostname;
* change the Zabbix agent hostname;
* regenerate PSK identity and key where required;
* verify IP addressing;
* update inventory;
* create a unique Zabbix host.

Example Agent 2 change:

```text
Hostname=AACP-BUD-SRV-02
```

Restart:

```bash
systemctl restart zabbix-agent2
```

A cloned VM using the original hostname may send data to the wrong Zabbix host.

---

## Host Availability Indicators

The frontend displays interface availability.

Possible states include:

* available;
* unavailable;
* unknown.

Availability is tracked separately for:

* Agent;
* SNMP;
* JMX;
* IPMI.

An unavailable interface does not always mean the host is completely offline.

Example:

```text
ICMP: Available
Agent: Unavailable
```

Possible causes:

* agent service stopped;
* firewall blocking port 10050;
* incorrect allowed server IP;
* TLS mismatch;
* agent interface misconfigured.

---

## Validate Passive Agent Connectivity

From the server or proxy:

```bash
zabbix_get \
    -s 192.168.10.20 \
    -p 10050 \
    -k agent.ping
```

Test hostname:

```bash
zabbix_get \
    -s 192.168.10.20 \
    -k system.hostname
```

Check the TCP port:

```bash
nc -zv 192.168.10.20 10050
```

If the TCP connection works but `zabbix_get` fails, review:

* agent permissions;
* `Server=` configuration;
* TLS settings;
* requested item key;
* agent logs.

---

## Validate Active Agent Connectivity

On the monitored host, review the agent log.

Linux:

```bash
journalctl -u zabbix-agent2 -n 100 --no-pager
```

Windows:

```text
C:\Program Files\Zabbix Agent 2\zabbix_agent2.log
```

Common errors:

```text
host not found
no active checks
cannot connect to server
connection rejected
TLS handshake failed
```

Verify:

```text
ServerActive=
Hostname=
TLSConnect=
TLSPSKIdentity=
TLSPSKFile=
```

---

## Validate SNMP Hosts

Test device reachability:

```bash
ping -c 4 192.168.10.1
```

Test UDP port behavior where possible:

```bash
nmap -sU -p 161 192.168.10.1
```

Test SNMP directly:

```bash
snmpget \
    -v2c \
    -c COMMUNITY_STRING \
    192.168.10.1 \
    1.3.6.1.2.1.1.3.0
```

Expected result:

```text
SNMPv2-MIB::sysUpTime.0
```

If testing works manually but not in Zabbix, verify:

* host interface;
* template;
* SNMP version;
* macros;
* proxy assignment;
* item OID;
* timeout settings.

---

## Host Maintenance

Maintenance mode suppresses or controls problem generation during planned work.

Use maintenance for:

* operating system updates;
* firmware upgrades;
* server reboots;
* network maintenance;
* planned shutdowns;
* application deployments.

Do not disable hosts for normal maintenance.

Maintenance periods are covered in a later chapter.

---

## Ownership and Responsibility

Each host should have a defined owner.

Ownership may be stored using:

* tags;
* inventory;
* description;
* host groups.

Example tags:

```text
owner=SARABEL
support_team=infrastructure
customer=AACP
```

Ownership helps determine:

* who receives notifications;
* who approves maintenance;
* who investigates incidents;
* who controls credentials;
* who authorizes decommissioning.

---

## Priority Classification

Hosts may be classified according to operational impact.

Example:

| Priority | Description                             |
| -------- | --------------------------------------- |
| Critical | Business operation stops if unavailable |
| High     | Major service degradation               |
| Medium   | Limited operational impact              |
| Low      | Non-critical or informational system    |

Example tag:

```text
priority=critical
```

Priority tags can control:

* escalation timing;
* notification channels;
* dashboard filters;
* SLA reporting;
* maintenance approval.

---

## Production and Test Hosts

Separate production and non-production hosts using tags or groups.

Example:

```text
environment=production
environment=test
environment=development
```

Do not rely only on host names such as:

```text
TEST-SRV
```

A consistent environment tag is easier to use in:

* actions;
* dashboards;
* maintenance;
* API queries;
* reporting.

---

## Documentation Standards

Each host should include enough information for another administrator to understand its purpose.

Recommended documentation:

```text
Purpose
Customer
Location
Owner
Primary services
Dependencies
Backup method
Maintenance window
Support contact
Replacement device
Special monitoring notes
```

Example description:

```text
Purpose:
Primary file server for AACP.

Dependencies:
- MikroTik router
- Synology NAS
- UPS-01

Backup:
Daily Veeam incremental backup.
Weekly full backup on Sunday.

Monitoring:
Zabbix Agent 2 through AACP-BUD-ZXP-01.
```

---

## Host Review Process

Hosts should be reviewed regularly.

Recommended review frequency:

| Environment             |   Review interval |
| ----------------------- | ----------------: |
| Critical infrastructure |           Monthly |
| Production servers      |         Quarterly |
| Network devices         |         Quarterly |
| Workstations            | Every 6–12 months |
| Disabled hosts          |           Monthly |
| Temporary hosts         |           Monthly |

Review:

* host status;
* ownership;
* templates;
* interfaces;
* macros;
* tags;
* inventory;
* proxy assignment;
* unsupported items;
* obsolete devices.

---

## Host Audit Queries

Useful frontend filters include:

```text
Hosts without templates
Disabled hosts
Hosts without interfaces
Hosts with unsupported items
Hosts without inventory
Hosts with duplicate visible names
Hosts assigned to the wrong proxy
Hosts without customer tags
```

The API may be used for more advanced audits.

---

## Common Host Management Mistakes

Avoid the following:

* inconsistent host names;
* missing host groups;
* assigning every host to one generic group;
* using visible names as technical identifiers;
* incorrect active agent hostname;
* duplicate IP addresses;
* incorrect proxy assignment;
* storing passwords in descriptions;
* manually duplicating template configuration;
* leaving default SNMP community strings;
* creating hosts without ownership;
* deleting decommissioned hosts immediately;
* failing to review cloned virtual machines;
* leaving disabled hosts indefinitely;
* assigning unnecessary interfaces;
* using inconsistent tags.

---

## Troubleshooting Host Data Collection

When a host has no data, follow a structured process.

### Step 1: Verify host status

Confirm:

```text
Host status: Enabled
```

### Step 2: Verify interface

Check:

* IP address;
* DNS name;
* port;
* default interface;
* interface type.

### Step 3: Verify proxy

Confirm the host is assigned to the expected server or proxy.

### Step 4: Test network connectivity

```bash
ping -c 4 <HOST_IP>
nc -zv <HOST_IP> 10050
```

### Step 5: Test the protocol

Agent:

```bash
zabbix_get -s <HOST_IP> -k agent.ping
```

SNMP:

```bash
snmpget -v2c -c <COMMUNITY> <HOST_IP> 1.3.6.1.2.1.1.3.0
```

### Step 6: Review templates

Confirm the required templates are linked.

### Step 7: Review macros

Check credentials and thresholds.

### Step 8: Review logs

Server:

```bash
tail -n 100 /var/log/zabbix/zabbix_server.log
```

Proxy:

```bash
tail -n 100 /var/log/zabbix/zabbix_proxy.log
```

Agent:

```bash
journalctl -u zabbix-agent2 -n 100 --no-pager
```

---

## Host Management Checklist

### Standards

* [ ] Naming convention documented
* [ ] Host group structure documented
* [ ] Tag vocabulary documented
* [ ] Inventory fields defined
* [ ] Proxy assignment rules defined
* [ ] Lifecycle process documented

### Per-host configuration

* [ ] Host name unique
* [ ] Visible name descriptive
* [ ] Correct groups assigned
* [ ] Correct interface configured
* [ ] Correct proxy assigned
* [ ] Required templates linked
* [ ] Host macros reviewed
* [ ] Tags configured
* [ ] Inventory configured
* [ ] Encryption enabled where required
* [ ] Description completed

### Validation

* [ ] Agent or SNMP reachable
* [ ] Latest data received
* [ ] Availability indicator valid
* [ ] Unsupported items reviewed
* [ ] Triggers reviewed
* [ ] Ownership assigned
* [ ] Backup monitoring assigned
* [ ] Decommission process defined

---

## Recommended Host Configuration Example

Example Windows application server:

```text
Host name:
AACP-BUD-SRV-01

Visible name:
AACP Primary Application Server

Host groups:
Customers/AACP/Servers
Operating Systems/Windows
Sites/Budapest
Environment/Production

Agent interface:
192.168.10.20
TCP 10050

Monitored by:
AACP-BUD-ZXP-01

Templates:
Windows by Zabbix agent active
ICMP Ping
Veeam Backup Monitoring
Windows Security Monitoring

Tags:
customer=AACP
site=Budapest
environment=production
device_type=server
priority=critical
owner=SARABEL

Macros:
{$CPU.UTIL.MAX}=90
{$DISK.FREE.MIN}=15
{$BACKUP.MAX.AGE}=86400

Inventory:
Automatic

Encryption:
PSK
```

---

## Recommended Network Device Example

Example MikroTik router:

```text
Host name:
AACP-BUD-FW-01

Visible name:
AACP Main MikroTik Router

Host groups:
Customers/AACP/Network
Network Devices/MikroTik
Sites/Budapest
Environment/Production

SNMP interface:
192.168.10.1
UDP 161

Monitored by:
AACP-BUD-ZXP-01

Templates:
MikroTik RouterOS by SNMP
ICMP Ping
WAN Availability Monitoring

Tags:
customer=AACP
site=Budapest
environment=production
device_type=firewall
vendor=MikroTik
priority=critical
owner=SARABEL

Macros:
{$SNMP.USER}=zabbix
{$SNMP.AUTH.PASSWORD}=Secret macro
{$SNMP.PRIV.PASSWORD}=Secret macro

Inventory:
Automatic and manually completed
```

---

## Summary

Host management defines how monitored systems are identified, organized and maintained.

A scalable host management model requires:

* consistent naming;
* structured host groups;
* correct interfaces;
* reusable templates;
* standardized tags;
* controlled macros;
* accurate inventory;
* proper proxy placement;
* encrypted communication;
* documented lifecycle procedures.

The next chapter explains how to create, link and maintain reusable Zabbix templates.

[Next: Templates →](05-templates.md)

