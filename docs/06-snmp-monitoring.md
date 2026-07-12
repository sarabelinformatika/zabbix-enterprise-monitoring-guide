# SNMP Monitoring

Simple Network Management Protocol remains one of the most widely used methods for monitoring network and infrastructure devices.

It is commonly used with:

* routers;
* switches;
* firewalls;
* wireless access points;
* UPS systems;
* storage appliances;
* printers;
* environmental sensors;
* power distribution units;
* server management controllers;
* telecommunications equipment.

SNMP provides standardized access to operational metrics without requiring a Zabbix agent on the monitored device.

A reliable SNMP design requires more than entering a community string and linking a template.

It must address:

* protocol version;
* authentication;
* encryption;
* network access;
* object identifiers;
* discovery;
* polling intervals;
* interface selection;
* device-specific behavior;
* template quality;
* performance;
* security.

---

## The Role of SNMP in Monitoring

SNMP allows a monitoring system to query structured management information from a device.

A typical polling workflow is:

```text
Zabbix Server or Proxy
          │
          │ SNMP request
          ▼
      Network Device
          │
          │ SNMP response
          ▼
Zabbix Server or Proxy
```

The monitoring platform requests a specific object identifier and the device returns the current value.

Typical values include:

* device uptime;
* CPU utilization;
* memory usage;
* interface status;
* interface traffic;
* packet errors;
* packet drops;
* temperature;
* voltage;
* fan state;
* power supply state;
* storage usage;
* battery condition.

SNMP is especially useful where installing an agent is impossible or unsupported.

---

## SNMP Architecture

A production SNMP architecture should define which component performs the polling.

Possible designs include:

```text
Central Zabbix Server
        │
        ▼
   Network Devices
```

or:

```text
Central Zabbix Server
        │
        ▼
   Zabbix Proxy
        │
        ▼
   Network Devices
```

The proxy-based model is usually preferable for remote sites.

Benefits include:

* local polling;
* reduced WAN traffic;
* local buffering;
* simplified firewall rules;
* improved fault isolation;
* better scalability.

---

## SNMP Versions

The main SNMP versions used in production are:

* SNMPv1;
* SNMPv2c;
* SNMPv3.

Each version provides different security and operational characteristics.

---

## SNMPv1

SNMPv1 is the original protocol version.

It provides:

* basic polling;
* basic trap support;
* community-string authentication;
* limited counter support.

Limitations include:

* no encryption;
* weak authentication;
* 32-bit counters;
* limited error handling;
* outdated security model.

SNMPv1 should be used only where required by legacy hardware.

---

## SNMPv2c

SNMPv2c is still widely used.

It provides:

* improved error handling;
* bulk operations;
* 64-bit counters;
* better performance than SNMPv1;
* community-string authentication.

Its primary weakness is security.

The community string is transmitted without modern encryption and acts similarly to a shared password.

SNMPv2c may be acceptable on:

* isolated management networks;
* tightly controlled VLANs;
* legacy environments;
* devices without SNMPv3 support.

It should not be exposed across untrusted networks.

---

## SNMPv3

SNMPv3 introduces a modern security model.

It can provide:

* username-based authentication;
* message integrity;
* encryption;
* replay protection;
* access control.

SNMPv3 should be the preferred version where supported.

The available security levels are:

| Security level | Authentication | Encryption |
| -------------- | -------------- | ---------- |
| noAuthNoPriv   | No             | No         |
| authNoPriv     | Yes            | No         |
| authPriv       | Yes            | Yes        |

For production environments, use:

```text
authPriv
```

where possible.

---

## SNMPv3 Authentication and Privacy

SNMPv3 commonly uses:

### Authentication algorithms

* MD5;
* SHA;
* SHA-224;
* SHA-256;
* SHA-384;
* SHA-512.

### Privacy algorithms

* DES;
* AES;
* AES-192;
* AES-256.

The exact algorithms depend on device support.

Use the strongest mutually supported algorithms.

Avoid:

* MD5;
* DES;
* weak shared passwords;
* default usernames.

A typical modern baseline is:

```text
Security level: authPriv
Authentication: SHA
Privacy: AES
```

Where supported, stronger SHA and AES variants may be used.

---

## SNMP Security Model

SNMP should be considered a management-plane protocol.

It may expose information such as:

* device model;
* serial number;
* firmware version;
* interface names;
* IP addressing;
* routing information;
* system contact;
* system location;
* hardware health;
* service state.

This information can help an attacker understand the infrastructure.

SNMP access should therefore be restricted by:

* source IP;
* management VLAN;
* firewall policy;
* access control lists;
* SNMP views;
* authentication;
* encryption.

---

## Read-Only Access

Monitoring should use read-only SNMP access.

Write access is rarely required for Zabbix monitoring.

A read-write community or SNMPv3 user may allow changes to:

* interface state;
* configuration;
* routing;
* system values;
* device behavior.

Recommended principle:

```text
Zabbix SNMP account = read-only
```

Never reuse administrative SNMP credentials for monitoring unless there is a documented technical requirement.

---

## SNMP Views

SNMP views restrict which object identifiers a user may access.

A view can permit:

* system information;
* interface statistics;
* hardware sensors;
* vendor-specific monitoring branches.

It can deny:

* configuration objects;
* sensitive data;
* write-capable objects;
* unrelated MIB trees.

A dedicated monitoring view reduces exposure if credentials are compromised.

---

## Network Placement

SNMP traffic should remain on a trusted management path.

Recommended design:

```text
                    Core Firewall
                         │
          ┌──────────────┴──────────────┐
          │                             │
          ▼                             ▼
   Production VLAN               Management VLAN
                                         │
                                         ▼
                                  Zabbix Proxy
                                         │
                                         ▼
                                  Network Devices
```

The Zabbix server or proxy should not require unrestricted access to all device management interfaces.

Only the required protocol should be allowed.

---

## Firewall Design

SNMP polling normally uses:

```text
UDP 161
```

SNMP traps normally use:

```text
UDP 162
```

A minimal rule set may look like:

```text
Zabbix Proxy → Network Device
UDP 161

Network Device → SNMP Trap Receiver
UDP 162
```

Recommended firewall policy:

* specify exact source addresses;
* specify exact destination addresses;
* allow only UDP 161 and UDP 162 where required;
* deny public Internet access;
* log unexpected management traffic;
* separate trap and polling flows;
* review obsolete rules.

---

## Polling and Traps

SNMP supports two different monitoring models:

* polling;
* traps.

They serve different purposes.

---

## SNMP Polling

With polling, Zabbix periodically requests values.

Examples:

* interface traffic;
* CPU utilization;
* memory usage;
* temperature;
* device uptime;
* power state.

Polling provides:

* predictable collection intervals;
* current state verification;
* historical trend data;
* consistent trigger evaluation.

Polling is the primary SNMP monitoring method.

---

## SNMP Traps

Traps are unsolicited messages sent by the device.

Examples:

* interface down;
* power supply failure;
* device restart;
* authentication failure;
* hardware fault;
* temperature alarm.

Traps provide faster event notification but have limitations.

A trap may be:

* lost;
* blocked;
* delayed;
* duplicated;
* malformed;
* unsupported by the receiving parser.

Traps should normally supplement polling.

They should not be the only source of truth for critical conditions.

---

## Polling as State, Traps as Events

A practical model is:

```text
Polling = current state
Traps = event notification
```

For example:

```text
Trap:
Interface changed to down

Polling:
Interface is currently down
```

The trap provides fast notification.

The poll confirms the persistent state.

---

## Management Information Base

A Management Information Base defines the structure and meaning of SNMP objects.

A MIB contains symbolic names for object identifiers.

Example:

```text
sysUpTime.0
```

may represent:

```text
1.3.6.1.2.1.1.3.0
```

MIB files improve readability but do not change the actual protocol communication.

The device and monitoring system exchange numerical OIDs.

---

## Object Identifiers

An object identifier uniquely identifies an SNMP value.

Example:

```text
1.3.6.1.2.1.1.3.0
```

This OID represents system uptime in standard SNMP implementations.

OID structure follows a hierarchy.

```text
iso
└── org
    └── dod
        └── internet
            └── mgmt
                └── mib-2
```

Vendor-specific objects are typically located under:

```text
1.3.6.1.4.1
```

This branch is known as the private enterprise tree.

---

## Standard and Vendor-Specific OIDs

Standard OIDs provide common metrics.

Examples include:

* system information;
* interface statistics;
* IP information;
* TCP statistics;
* UDP statistics.

Vendor-specific OIDs provide additional details such as:

* proprietary hardware sensors;
* model-specific health data;
* wireless metrics;
* RAID controller status;
* device licensing;
* vendor-defined CPU values;
* custom interface states.

A complete template often uses both.

---

## Scalar and Tabular OIDs

SNMP values are commonly divided into:

* scalar objects;
* table objects.

### Scalar object

A scalar represents one value.

Example:

```text
sysUpTime.0
```

### Table object

A table contains multiple indexed rows.

Example interface table:

```text
ifDescr.1
ifDescr.2
ifDescr.3
```

The index identifies a specific interface.

Tables are typically handled using low-level discovery.

---

## SNMP Indexes

SNMP tables use indexes to identify rows.

An interface may have:

```text
ifIndex=5
```

Its related values may be:

```text
ifDescr.5
ifOperStatus.5
ifInOctets.5
ifOutOctets.5
```

Indexes may change after:

* reboot;
* firmware upgrade;
* hardware replacement;
* interface reconfiguration;
* module replacement.

Templates should avoid assuming that a fixed interface always uses a fixed index.

Discovery should map interfaces dynamically.

---

## SNMP Walk

An SNMP walk queries a sequence of OIDs.

Example:

```bash
snmpwalk -v2c \
    -c COMMUNITY_STRING \
    192.168.10.1 \
    1.3.6.1.2.1.1
```

This retrieves the system branch.

A full walk may produce a large amount of data.

Use targeted walks rather than walking the entire device unnecessarily.

---

## SNMP Get

An SNMP get requests a single value.

Example:

```bash
snmpget -v2c \
    -c COMMUNITY_STRING \
    192.168.10.1 \
    1.3.6.1.2.1.1.3.0
```

This is useful for validating:

* credentials;
* connectivity;
* OID support;
* returned data type;
* expected value.

---

## SNMP Bulk Operations

SNMPv2c and SNMPv3 support bulk operations.

Bulk requests improve efficiency when retrieving tables.

They reduce:

* packet count;
* request overhead;
* polling time;
* device load;
* network latency effects.

Zabbix can use bulk processing for suitable SNMP items.

However, some devices implement bulk operations poorly.

If a device returns incomplete or incorrect results, bulk requests may need to be disabled for that host or template.

---

## Device Capability Assessment

Before building a template, determine:

* supported SNMP versions;
* supported authentication algorithms;
* supported privacy algorithms;
* available MIBs;
* standard OID behavior;
* vendor-specific OIDs;
* counter width;
* interface index stability;
* maximum request size;
* firmware limitations;
* rate limits.

Device documentation should be treated as the primary source.

Do not assume that two models from the same vendor expose identical objects.

---

## SNMP Credentials

Credentials should be unique by trust boundary.

Avoid using one global community string for every customer and site.

A better model is:

```text
Customer A SNMP credentials
Customer B SNMP credentials
Internal infrastructure credentials
Laboratory credentials
```

For SNMPv3, use separate users where operationally feasible.

This reduces the impact of credential exposure.

---

## Community String Design

For SNMPv2c, community strings should:

* be long;
* be random;
* be unique;
* avoid dictionary words;
* avoid device names;
* avoid customer names;
* never use defaults.

Unsafe examples:

```text
public
private
monitoring
zabbix
companyname
```

Community strings should be stored using secret macros.

---

## Macro-Based Credential Management

Templates should not contain real credentials.

Use macros such as:

```text
{$SNMP_COMMUNITY}
{$SNMP.USER}
{$SNMP.AUTH.PASSWORD}
{$SNMP.PRIV.PASSWORD}
```

Credential values may be assigned at:

* global level;
* template level;
* host level.

The host or customer level is usually safer for multi-tenant environments.

---

## SNMPv3 Macro Model

A standardized macro set may include:

```text
{$SNMPV3.USER}
{$SNMPV3.SECURITYLEVEL}
{$SNMPV3.AUTHPROTOCOL}
{$SNMPV3.AUTHPASSPHRASE}
{$SNMPV3.PRIVPROTOCOL}
{$SNMPV3.PRIVPASSPHRASE}
{$SNMPV3.CONTEXT}
```

Macro names should remain consistent across templates.

This simplifies host onboarding and automation.

---

## Host Interface Design

Every SNMP host requires an SNMP interface.

Example:

```text
Type:
SNMP

IP address:
192.168.10.1

Port:
161
```

The selected address must be reachable from the assigned Zabbix server or proxy.

Use the management IP rather than a production service IP where possible.

---

## Management IP Selection

The SNMP interface should use a stable management address.

Preferred options:

* dedicated management VLAN address;
* loopback address;
* out-of-band management address;
* stable internal interface.

Avoid using:

* dynamic WAN addresses;
* temporary tunnel addresses;
* client-facing service addresses;
* addresses that depend on unstable routing.

The monitored address should represent the management identity of the device.

---

## Proxy Assignment

A remote SNMP device should normally be monitored by a local proxy.

Example:

```text
Central Zabbix Server
        │
        ▼
Customer Proxy
        │
        ▼
MikroTik Router
```

Benefits include:

* low-latency polling;
* local reachability;
* fewer VPN dependencies;
* reduced central firewall exposure;
* buffering during outages.

The proxy must have access to the management address of the device.

---

## Template Selection

A template should match:

* vendor;
* device family;
* firmware;
* SNMP version;
* available OIDs;
* operational purpose.

Do not link multiple broad SNMP templates without checking for overlap.

Possible conflicts include:

* duplicate interface discovery;
* duplicate uptime items;
* duplicate availability triggers;
* duplicate traffic graphs;
* duplicate item keys.

---

## Official Templates

Official Zabbix templates are useful starting points.

They may include:

* standard interface discovery;
* device health monitoring;
* hardware sensors;
* trigger logic;
* macros;
* value maps;
* graphs.

Before production use, review:

* item count;
* discovery filters;
* trigger thresholds;
* unsupported items;
* polling intervals;
* required macros;
* firmware compatibility.

Official does not automatically mean optimal for every environment.

---

## Custom Vendor Templates

A custom template may be required when:

* no official template exists;
* official coverage is incomplete;
* vendor OIDs are proprietary;
* local operational requirements differ;
* specific hardware sensors are critical;
* only selected metrics are required.

Custom templates should be:

* documented;
* versioned;
* tested;
* exported;
* reviewed after firmware changes.

---

## Interface Discovery

Interface discovery is one of the most common SNMP use cases.

It typically discovers:

* interface index;
* interface name;
* interface description;
* interface alias;
* interface type;
* administrative state;
* operational state;
* speed;
* physical address.

A discovery rule may then create:

* inbound traffic item;
* outbound traffic item;
* error counters;
* packet drop counters;
* status triggers;
* traffic graphs.

---

## Interface Naming

Devices may expose several identifiers:

```text
ifName
ifDescr
ifAlias
ifIndex
```

Their meaning varies by platform.

Example:

```text
ifName:
ether1

ifDescr:
ether1

ifAlias:
WAN uplink

ifIndex:
1
```

Templates should display the identifier most useful to administrators.

Often the best display format combines technical and descriptive values.

Example:

```text
Interface ether1: WAN uplink
```

---

## Interface Filtering

Not every discovered interface should be monitored.

Common exclusions include:

* loopback interfaces;
* null interfaces;
* temporary tunnel interfaces;
* inactive virtual adapters;
* internal bridge members;
* test interfaces;
* dynamic VPN interfaces;
* interfaces without operational importance.

Discovery filters should be based on:

* interface name;
* alias;
* type;
* administrative state;
* regular expressions.

---

## Discovery Overrides

Overrides allow different behavior for selected interfaces.

Examples:

* disable status triggers on intentionally unused ports;
* increase severity for WAN interfaces;
* use lower traffic thresholds on low-bandwidth links;
* disable graphs on virtual interfaces;
* retain monitoring for administratively down but critical ports.

Overrides should be preferred over duplicating entire templates.

---

## Interface Status Monitoring

SNMP commonly exposes:

```text
ifAdminStatus
ifOperStatus
```

### Administrative status

Represents the configured state.

Typical values:

```text
up
down
testing
```

### Operational status

Represents the actual state.

Typical values:

```text
up
down
testing
unknown
dormant
notPresent
lowerLayerDown
```

A useful trigger often compares both values.

Example logic:

```text
Administrative state = up
AND
Operational state = down
```

This avoids alerts for intentionally disabled interfaces.

---

## Interface Traffic Counters

Traffic monitoring uses byte or octet counters.

Common objects include:

```text
ifInOctets
ifOutOctets
ifHCInOctets
ifHCOutOctets
```

The `HC` counters are 64-bit high-capacity counters.

They should be preferred for high-speed interfaces.

---

## Counter Wrap

32-bit counters wrap quickly on fast interfaces.

At high traffic rates, a 32-bit counter may reset within minutes.

This can produce:

* inaccurate graphs;
* negative deltas;
* false traffic spikes;
* lost utilization data.

Use 64-bit counters where supported.

---

## Traffic Rate Calculation

SNMP counters usually return cumulative values.

Zabbix converts them into rates using change-per-second preprocessing.

Conceptually:

```text
Current counter - Previous counter
----------------------------------
        Time difference
```

The result may be stored as:

```text
bits per second
```

When the device returns bytes, multiply by eight to calculate bits.

---

## Interface Speed

Interface utilization requires a reliable speed value.

Common sources include:

```text
ifSpeed
ifHighSpeed
```

Potential problems include:

* incorrect speed reporting;
* zero values;
* wireless rate variation;
* bonded interface behavior;
* virtual interfaces;
* tunnel interfaces;
* auto-negotiation changes.

Utilization triggers should not rely blindly on device-reported speed.

---

## Error and Drop Counters

Important interface quality metrics include:

* inbound errors;
* outbound errors;
* inbound discards;
* outbound discards;
* CRC errors;
* collisions;
* queue drops.

A rising counter may indicate:

* cable problems;
* duplex mismatch;
* hardware failure;
* congestion;
* overloaded buffers;
* wireless interference;
* provider issues.

Triggers should normally evaluate counter rate or change, not the cumulative total.

---

## Device Availability

SNMP availability should be combined with ICMP where appropriate.

Example:

```text
ICMP unavailable
SNMP unavailable
```

These conditions provide different information.

Possible states:

| ICMP | SNMP | Interpretation                       |
| ---- | ---- | ------------------------------------ |
| Up   | Up   | Device reachable and SNMP available  |
| Up   | Down | Device reachable, SNMP failure       |
| Down | Up   | ICMP blocked or inconsistent routing |
| Down | Down | Device or path likely unavailable    |

This distinction improves troubleshooting.

---

## Device Uptime

System uptime is commonly monitored using:

```text
sysUpTime
```

A sudden decrease may indicate:

* reboot;
* power interruption;
* firmware upgrade;
* management service restart.

Some devices reset SNMP uptime without a full system reboot.

Vendor-specific uptime values may provide more accurate results.

---

## CPU and Memory Monitoring

CPU and memory OIDs vary widely between vendors.

Potential issues include:

* one-minute versus five-minute averages;
* percentage versus raw units;
* multiple CPU cores;
* unavailable total memory;
* inconsistent used and free definitions;
* firmware-specific OIDs.

Validate returned values against the device's own management interface.

---

## Hardware Health

Vendor-specific MIBs may expose:

* temperature;
* fan speed;
* voltage;
* power supply status;
* storage state;
* battery health;
* sensor alarms.

These items should use:

* value maps;
* clear units;
* documented thresholds;
* trigger dependencies;
* meaningful severities.

A hardware state value such as `3` is not useful without interpretation.

---

## Temperature Monitoring

Temperature thresholds should account for:

* sensor type;
* device location;
* vendor limits;
* warning and critical values;
* ambient conditions;
* measurement unit.

Do not apply one global temperature threshold to all devices.

A CPU sensor and ambient rack sensor require different limits.

Use macros or sensor-specific discovery contexts.

---

## Wireless Monitoring

Wireless devices may expose:

* signal strength;
* signal-to-noise ratio;
* channel utilization;
* connected client count;
* retry rate;
* noise floor;
* radio state;
* channel;
* frequency;
* transmit power.

Wireless metrics can change rapidly.

Polling intervals should balance:

* diagnostic value;
* device load;
* database volume;
* operational need.

---

## MikroTik Monitoring

MikroTik RouterOS devices are commonly monitored using SNMP.

Typical metrics include:

* device uptime;
* CPU load;
* free memory;
* storage usage;
* interface traffic;
* interface status;
* wireless registration;
* temperature;
* voltage;
* board information;
* RouterOS version.

A typical architecture is:

```text
Zabbix Proxy
      │
      │ SNMPv3
      ▼
MikroTik Router
```

Use a dedicated read-only SNMPv3 user where supported.

---

## MikroTik Design Considerations

MikroTik environments may contain:

* physical Ethernet ports;
* bridges;
* VLAN interfaces;
* WireGuard interfaces;
* PPP interfaces;
* dynamic tunnels;
* bonding interfaces;
* wireless interfaces.

Monitoring every interface may generate unnecessary data.

Discovery filters should exclude or classify:

* dynamic PPP sessions;
* temporary tunnels;
* inactive interfaces;
* internal-only bridge members;
* interfaces without operational relevance.

WAN, LAN and VPN interfaces may require different alert policies.

---

## WAN Monitoring

WAN monitoring should combine several metrics.

Recommended signals:

* interface operational state;
* ICMP reachability to gateway;
* external reachability;
* latency;
* packet loss;
* traffic utilization;
* error counters;
* public IP change where relevant.

A WAN link should not be considered healthy only because the physical interface is up.

Example:

```text
Physical interface: Up
Provider gateway: Reachable
External target: Unreachable
```

This may indicate an upstream routing or provider problem.

---

## Redundant Links

For redundant WAN or uplink designs, alerts should reflect service impact.

Example:

```text
WAN1 failed
WAN2 active
Service remains available
```

This may be:

```text
Severity: Warning
```

If both links fail:

```text
Severity: Disaster
```

Trigger dependencies and service-level logic should prevent unnecessary critical alarms during successful failover.

---

## SNMP Timeout Design

SNMP uses UDP and does not guarantee delivery.

Timeouts may occur because of:

* packet loss;
* high latency;
* device CPU load;
* rate limiting;
* firewall behavior;
* oversized responses;
* unsupported OIDs;
* proxy overload.

Timeout values should not be increased blindly.

First determine whether the problem is:

* network-related;
* device-related;
* template-related;
* capacity-related.

---

## Retries and Polling Load

Frequent retries can amplify load during outages.

When hundreds of devices stop responding, the monitoring system may spend significant resources waiting for timeouts.

This can create:

* poller saturation;
* queue growth;
* delayed checks;
* false secondary problems.

Capacity planning must include failure scenarios, not only normal operation.

---

## SNMP Poller Capacity

Monitor Zabbix internal metrics such as:

* SNMP poller utilization;
* poller queue;
* unreachable poller utilization;
* preprocessing utilization;
* values processed per second.

Persistently high utilization may require:

* more poller processes;
* longer polling intervals;
* additional proxies;
* template optimization;
* reduced item count;
* bulk request tuning.

Process tuning should follow measurement.

---

## Polling Interval Strategy

Recommended example intervals:

| Metric              |      Interval |
| ------------------- | ------------: |
| Device availability | 30–60 seconds |
| Interface state     |    60 seconds |
| Interface traffic   |    60 seconds |
| CPU and memory      |   1–5 minutes |
| Hardware sensors    |   1–5 minutes |
| Device inventory    |   12–24 hours |
| Serial number       |      24 hours |
| Firmware version    |     1–6 hours |

Static values should not be polled frequently.

---

## Polling Synchronization

If thousands of items run at the same exact interval, they may create polling bursts.

Example:

```text
All items run every 60 seconds
```

This can cause periodic load spikes.

Flexible intervals, preprocessing and proxy distribution can help smooth the workload.

Large deployments should review queue behavior over time.

---

## SNMP Preprocessing

SNMP values may require preprocessing.

Common operations include:

* multiplier;
* change per second;
* discard unchanged;
* value mapping;
* regular expression extraction;
* JavaScript conversion;
* hexadecimal conversion;
* text normalization.

Example:

```text
Raw counter:
123456789 bytes

Preprocessing:
Change per second
Multiply by 8

Stored value:
bits per second
```

---

## Unsupported Values

A device may return:

```text
noSuchObject
noSuchInstance
endOfMibView
```

Possible causes:

* unsupported OID;
* incorrect index;
* firmware difference;
* restricted SNMP view;
* wrong template;
* device mode;
* disabled feature.

Unsupported values should be investigated rather than permanently ignored.

---

## Firmware Changes

Firmware upgrades may change:

* OID availability;
* table indexes;
* data types;
* returned values;
* vendor MIB behavior;
* authentication support.

After a firmware upgrade, validate:

* critical items;
* interface discovery;
* hardware sensors;
* trigger state;
* unsupported item count;
* trap behavior.

Monitoring is part of firmware change validation.

---

## Device Replacement

When replacing a device, consider whether to:

* reuse the existing Zabbix host;
* create a new host;
* preserve historical data;
* update inventory;
* regenerate credentials;
* verify interface discovery;
* reset expected serial number.

Reusing a host preserves historical continuity but may mix data from different hardware.

Creating a new host provides clearer asset separation.

The decision should follow asset and audit requirements.

---

## SNMP Traps Architecture

A trap-processing architecture typically includes:

```text
Network Device
      │
      │ UDP 162
      ▼
SNMP Trap Daemon
      │
      ▼
Trap Processing
      │
      ▼
Zabbix Server
```

Common components include:

* snmptrapd;
* trap log files;
* trap receiver scripts;
* Zabbix trapper items;
* preprocessing rules.

Trap processing should be tested end to end.

---

## Trap Source Validation

Trap receivers should accept messages only from authorized sources.

Controls may include:

* firewall restrictions;
* source address validation;
* SNMPv3 trap authentication;
* separate management VLAN;
* controlled trap communities.

An unrestricted trap receiver can be abused to generate:

* false alarms;
* log flooding;
* processing load;
* misleading events.

---

## Trap Normalization

Vendor traps vary significantly.

A normalization layer may be required to convert different formats into consistent events.

Example:

```text
Vendor A:
linkDown

Vendor B:
portStatusChanged

Normalized event:
Interface operational state changed to down
```

Normalization improves:

* alert consistency;
* ticket classification;
* reporting;
* automation.

---

## Trap Deduplication

A single device failure may generate many traps.

Examples:

* interface down;
* neighbor lost;
* route removed;
* service unavailable;
* failover initiated.

Without correlation, one incident may generate many alerts.

Use:

* trigger dependencies;
* event correlation;
* trap preprocessing;
* time-based suppression;
* root-cause logic.

---

## Trap Retention

Raw trap logs may contain valuable diagnostic information.

Retention should consider:

* storage capacity;
* incident investigation;
* security requirements;
* compliance;
* device event volume.

Trap logs should be rotated and protected from uncontrolled growth.

---

## Testing SNMPv2c

Example connectivity test:

```bash
snmpget \
    -v2c \
    -c COMMUNITY_STRING \
    192.168.10.1 \
    1.3.6.1.2.1.1.3.0
```

Example system walk:

```bash
snmpwalk \
    -v2c \
    -c COMMUNITY_STRING \
    192.168.10.1 \
    1.3.6.1.2.1.1
```

Example interface names:

```bash
snmpwalk \
    -v2c \
    -c COMMUNITY_STRING \
    192.168.10.1 \
    1.3.6.1.2.1.31.1.1.1.1
```

---

## Testing SNMPv3

Example:

```bash
snmpget \
    -v3 \
    -l authPriv \
    -u zabbix \
    -a SHA \
    -A AUTH_PASSWORD \
    -x AES \
    -X PRIVACY_PASSWORD \
    192.168.10.1 \
    1.3.6.1.2.1.1.3.0
```

Example walk:

```bash
snmpwalk \
    -v3 \
    -l authPriv \
    -u zabbix \
    -a SHA \
    -A AUTH_PASSWORD \
    -x AES \
    -X PRIVACY_PASSWORD \
    192.168.10.1 \
    1.3.6.1.2.1.1
```

Avoid entering real credentials into shell history on shared systems.

---

## Packet-Level Troubleshooting

Use packet capture when required.

Example:

```bash
tcpdump -ni any udp port 161
```

For traps:

```bash
tcpdump -ni any udp port 162
```

Packet capture can confirm:

* request leaves the server;
* response returns;
* source address is correct;
* firewall translation is occurring;
* packets are fragmented;
* retries are occurring.

Encrypted SNMPv3 payloads will not reveal the monitored value.

---

## Common SNMP Failure Modes

### Timeout

Possible causes:

* device unreachable;
* UDP 161 blocked;
* incorrect source address;
* SNMP disabled;
* rate limiting;
* device overloaded.

### Authentication failure

Possible causes:

* wrong community;
* wrong username;
* wrong authentication password;
* wrong privacy password;
* algorithm mismatch;
* incorrect security level.

### No such object

Possible causes:

* unsupported OID;
* wrong firmware;
* restricted SNMP view;
* incorrect table index.

### Incomplete walk

Possible causes:

* bulk implementation problem;
* response size limit;
* device timeout;
* packet fragmentation;
* MIB inconsistency.

---

## Troubleshooting Sequence

Use a structured process.

### Step 1: Verify reachability

```bash
ping -c 4 <DEVICE_IP>
```

### Step 2: Verify routing

```bash
ip route get <DEVICE_IP>
```

### Step 3: Test SNMP manually

```bash
snmpget ...
```

### Step 4: Verify credentials

Confirm:

* version;
* community or username;
* security level;
* algorithms;
* passwords;
* context.

### Step 5: Verify Zabbix interface

Confirm:

* IP address;
* port;
* proxy;
* SNMP version;
* macros.

### Step 6: Review Zabbix logs

```bash
tail -n 100 /var/log/zabbix/zabbix_server.log
```

or:

```bash
tail -n 100 /var/log/zabbix/zabbix_proxy.log
```

### Step 7: Capture traffic

```bash
tcpdump -ni any host <DEVICE_IP> and udp port 161
```

---

## Monitoring Through NAT

SNMP through NAT may work, but it introduces complexity.

Potential problems include:

* overlapping private addresses;
* port translation;
* source address mismatch;
* trap return path;
* multiple devices behind one address;
* firewall state timeout.

A local Zabbix proxy is usually preferable.

```text
Central Zabbix Server
        │
        ▼
Remote Zabbix Proxy
        │
        ▼
Private SNMP Devices
```

This avoids exposing UDP 161 across the Internet.

---

## Monitoring Through VPN

A site-to-site VPN is suitable when:

* routing is stable;
* management subnets are reachable;
* source IPs are predictable;
* firewall rules are controlled;
* latency is acceptable.

For larger sites, use a local proxy even when a VPN exists.

The proxy provides buffering and reduces central dependencies.

---

## Multi-Tenant SNMP Monitoring

Managed service environments should isolate customer SNMP access.

Recommended model:

```text
Customer A
├── Dedicated proxy
├── Customer-specific credentials
└── Customer-specific host groups

Customer B
├── Dedicated proxy
├── Customer-specific credentials
└── Customer-specific host groups
```

Avoid:

* one shared community across customers;
* direct central access to all management VLANs;
* shared proxy credentials without separation;
* global secret macros containing customer passwords.

---

## Inventory Through SNMP

SNMP may populate inventory fields such as:

* model;
* serial number;
* firmware version;
* vendor;
* system name;
* system contact;
* system location.

Automatically collected inventory should be validated.

Some devices return:

* empty values;
* generic model names;
* incorrect serial numbers;
* chassis serial instead of module serial;
* default location strings.

Manual correction may still be required.

---

## Monitoring Quality Validation

A device should not be considered fully onboarded until the following are verified:

* SNMP authentication works;
* critical OIDs return valid data;
* interface discovery is correct;
* unwanted interfaces are excluded;
* traffic graphs are realistic;
* status triggers behave correctly;
* hardware sensors are supported;
* device uptime is reliable;
* inventory is populated;
* proxy assignment is correct;
* no critical items are unsupported.

---

## Baseline Creation

After onboarding, establish a performance baseline.

Review:

* normal CPU usage;
* normal memory usage;
* typical interface traffic;
* expected packet loss;
* normal temperature range;
* usual client count;
* normal error counter growth.

Thresholds should be based on actual operating conditions where possible.

---

## Alert Tuning

SNMP templates frequently generate noise if left at default settings.

Common sources include:

* unused interfaces;
* temporary VPN interfaces;
* low-bandwidth links;
* expected packet errors;
* variable wireless metrics;
* unsupported sensors;
* transient device load.

Alert tuning should not simply disable problems.

It should identify whether the cause is:

* incorrect threshold;
* irrelevant entity;
* unstable metric;
* poor dependency design;
* actual infrastructure fault.

---

## SNMP Change Management

Changes to SNMP configuration should be documented.

Examples:

* credential rotation;
* protocol upgrade;
* template replacement;
* firmware upgrade;
* discovery filter modification;
* proxy reassignment;
* management IP change.

A change record should include:

* affected devices;
* old configuration;
* new configuration;
* test method;
* rollback plan;
* validation result.

---

## Credential Rotation

SNMP credentials should be rotated periodically and after:

* staff changes;
* suspected exposure;
* customer offboarding;
* device replacement;
* security incidents;
* configuration export leakage.

For large environments, use a staged process:

```text
Create new credential
→ Configure device
→ Update Zabbix
→ Validate monitoring
→ Remove old credential
```

Avoid changing every device simultaneously without validation.

---

## Monitoring the SNMP Layer

The SNMP monitoring system itself should be monitored.

Important signals include:

* SNMP poller utilization;
* timeout rate;
* unsupported item count;
* queue size;
* proxy availability;
* trap receiver availability;
* trap log growth;
* credential failure rate;
* polling latency.

A widespread increase in SNMP timeouts may indicate a monitoring platform issue rather than many simultaneous device failures.

---

## Recommended SNMP Architecture

A scalable reference model:

```text
                         Central Zabbix Server
                                  │
                                  ▼
                           Zabbix Database
                                  │
             ┌────────────────────┼────────────────────┐
             │                    │                    │
             ▼                    ▼                    ▼
        Site A Proxy         Site B Proxy         Customer Proxy
             │                    │                    │
       ┌─────┼─────┐        ┌─────┼─────┐        ┌─────┼─────┐
       │     │     │        │     │     │        │     │     │
       ▼     ▼     ▼        ▼     ▼     ▼        ▼     ▼     ▼
    Router Switch UPS    Router Switch AP      Router NAS  Firewall
```

Each proxy should poll local devices using the strongest supported SNMP version.

---

## Recommended Security Baseline

Use the following baseline where technically possible:

```text
SNMP version:
SNMPv3

Security level:
authPriv

Authentication:
SHA or stronger

Privacy:
AES or stronger

Access:
Read-only

Source restriction:
Zabbix proxy IP only

Network:
Dedicated management VLAN

Credentials:
Unique per customer or trust boundary
```

For legacy SNMPv2c devices:

```text
Use random community strings
Restrict by source IP
Use isolated management networks
Avoid Internet exposure
Plan replacement or protocol upgrade
```

---

## SNMP Onboarding Checklist

### Device preparation

* [ ] Device management IP is stable
* [ ] SNMP enabled
* [ ] SNMP version selected
* [ ] Read-only access configured
* [ ] Source IP restricted
* [ ] SNMP view configured where supported
* [ ] Credentials stored securely
* [ ] Firewall rule implemented

### Zabbix configuration

* [ ] Host created
* [ ] SNMP interface configured
* [ ] Correct proxy assigned
* [ ] Correct template linked
* [ ] Required macros configured
* [ ] Credentials use secret macros
* [ ] Tags configured
* [ ] Inventory mode selected

### Validation

* [ ] Manual SNMP query succeeds
* [ ] Device uptime received
* [ ] Interface discovery completed
* [ ] Critical interfaces identified
* [ ] Unwanted interfaces filtered
* [ ] Traffic values validated
* [ ] Hardware sensors validated
* [ ] Unsupported items reviewed
* [ ] Trigger behavior tested
* [ ] Graphs reviewed

---

## Template Review Checklist

### Security

* [ ] No credentials embedded
* [ ] SNMPv3 supported
* [ ] Read-only access assumed
* [ ] No write operations used
* [ ] Sensitive OIDs avoided where unnecessary

### Performance

* [ ] Item count justified
* [ ] Polling intervals reviewed
* [ ] Bulk behavior tested
* [ ] Discovery filters configured
* [ ] Static items use long intervals
* [ ] Proxy load estimated

### Quality

* [ ] OIDs documented
* [ ] Units correct
* [ ] Counter preprocessing correct
* [ ] Value maps configured
* [ ] Trigger thresholds use macros
* [ ] Interface status logic is accurate
* [ ] Firmware compatibility documented

---

## Common SNMP Mistakes

Avoid the following:

* using `public` as the community string;
* exposing UDP 161 to the Internet;
* using SNMPv1 without necessity;
* assigning read-write access;
* monitoring all interfaces without filtering;
* relying only on traps;
* using 32-bit counters on high-speed links;
* polling static values every minute;
* hard-coding credentials in templates;
* ignoring unsupported items;
* applying identical thresholds to all sensors;
* using one shared credential for every customer;
* monitoring remote devices directly when a proxy is available;
* assuming firmware upgrades do not affect OIDs;
* failing to validate values against the device interface.

---

## Enterprise Recommendations

For enterprise SNMP monitoring:

* use SNMPv3 with authentication and encryption;
* restrict access to dedicated monitoring sources;
* use local proxies for remote sites;
* separate customer credentials;
* use read-only accounts;
* standardize macro names;
* validate vendor templates;
* filter discovery aggressively;
* prefer 64-bit interface counters;
* combine traps with polling;
* monitor the SNMP poller layer;
* test after firmware changes;
* store custom templates in version control;
* document OID and MIB dependencies.

---

## Summary

SNMP provides scalable monitoring for network and infrastructure devices, but its reliability depends on architecture, security and template quality.

A strong SNMP implementation uses:

* secure protocol versions;
* restricted read-only access;
* local proxy polling;
* validated OIDs;
* controlled interface discovery;
* appropriate polling intervals;
* 64-bit counters;
* structured credentials;
* tested trigger logic;
* documented vendor behavior.

The next chapter explains how to deploy and operate Zabbix agent-based monitoring for Linux and Windows systems.

[Next: Agent Monitoring →](07-agent-monitoring.md)
