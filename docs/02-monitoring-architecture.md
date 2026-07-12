# Monitoring Architecture

A reliable monitoring platform is not defined only by the software installed on the monitoring server.

The complete architecture includes:

* monitored devices;
* monitored services;
* data collection methods;
* network communication paths;
* Zabbix server components;
* database services;
* proxies;
* agents;
* notification channels;
* operational and security controls.

A properly designed architecture ensures that monitoring remains scalable, secure and maintainable as the infrastructure grows.

---

## Architecture Overview

A typical Zabbix deployment consists of a central Zabbix server that collects, processes and stores monitoring data.

The monitored environment may include:

* physical servers;
* virtual machines;
* network devices;
* firewalls;
* routers;
* switches;
* wireless access points;
* storage systems;
* hypervisors;
* containers;
* websites;
* databases;
* cloud services;
* business applications.

The following diagram represents a basic centralized monitoring architecture:

```text
                          Administrators
                                │
                                ▼
                       Zabbix Web Interface
                                │
                                ▼
                         Zabbix Server
                                │
                    ┌───────────┴───────────┐
                    │                       │
                    ▼                       ▼
              Zabbix Database        Notification Systems
                    │                 E-mail / SMS / Webhook
                    │
                    ▼
          Monitoring Data and History

                                ▲
                                │
        ┌───────────────────────┼────────────────────────┐
        │                       │                        │
        ▼                       ▼                        ▼
   Zabbix Agents              SNMP Devices          Web Services
 Windows / Linux        Routers / Switches / UPS     HTTP / HTTPS
        │                       │                        │
        └───────────────────────┴────────────────────────┘
                                │
                                ▼
                         Monitored Network
```

In smaller environments, all Zabbix components may run on one server.

In larger environments, the server, database, frontend and proxy components should be separated according to performance, availability and security requirements.

---

## Core Zabbix Components

A Zabbix monitoring platform is built from several cooperating components.

The primary components are:

| Component       | Purpose                                                    |
| --------------- | ---------------------------------------------------------- |
| Zabbix Server   | Central processing and monitoring engine                   |
| Zabbix Database | Stores configuration, history, trends and events           |
| Zabbix Frontend | Web-based administration and visualization interface       |
| Zabbix Agent    | Collects operating system and application metrics          |
| Zabbix Proxy    | Collects data from remote or isolated environments         |
| Java Gateway    | Monitors Java applications using JMX                       |
| Zabbix Sender   | Sends custom monitoring data to Zabbix                     |
| Zabbix Get      | Tests and queries Zabbix agent items                       |
| Web Service     | Provides scheduled report generation and related functions |

Each component has a clearly defined role within the monitoring architecture.

---

## Zabbix Server

The Zabbix server is the central component of the monitoring environment.

It is responsible for:

* collecting monitoring data;
* polling monitored devices;
* receiving active agent data;
* processing item values;
* evaluating trigger expressions;
* detecting problems;
* generating events;
* executing actions;
* sending notifications;
* processing discovery rules;
* communicating with proxies;
* storing data in the database.

The Zabbix server does not normally store monitoring data directly in local files. Configuration and collected data are written to the configured database backend.

A failure of the Zabbix server does not affect the monitored infrastructure itself, but monitoring, event processing and notifications may stop until the service is restored.

For this reason, the Zabbix server should be treated as a critical infrastructure component.

---

## Zabbix Database

The database stores nearly all persistent information used by Zabbix.

This includes:

* hosts;
* host groups;
* templates;
* items;
* triggers;
* users;
* dashboards;
* events;
* alerts;
* historical values;
* trend data;
* audit records;
* discovery information;
* maintenance periods;
* configuration settings.

Supported database platforms depend on the installed Zabbix version and deployment model.

Common database platforms include:

* PostgreSQL;
* PostgreSQL with TimescaleDB;
* MySQL;
* MariaDB.

The database is frequently the most resource-intensive component of a Zabbix deployment.

Database performance directly affects:

* frontend response time;
* history queries;
* dashboard loading;
* report generation;
* housekeeping;
* event processing;
* overall server performance.

A production database should use reliable storage with low latency and sufficient write performance.

---

## Zabbix Frontend

The Zabbix frontend provides the graphical management interface.

Administrators use it to:

* create and manage hosts;
* assign templates;
* configure items;
* configure triggers;
* view problems;
* analyze historical data;
* create dashboards;
* configure users and roles;
* manage discovery;
* configure notifications;
* schedule maintenance;
* review audit information.

The frontend is a PHP-based web application typically served by:

* Nginx;
* Apache HTTP Server.

The frontend communicates directly with the Zabbix database.

It does not normally communicate directly with monitored hosts.

```text
Administrator
      │
      ▼
Web Browser
      │
      ▼
Nginx or Apache
      │
      ▼
Zabbix Frontend
      │
      ▼
Zabbix Database
```

Access to the frontend should always be protected using HTTPS.

---

## Zabbix Agent

The Zabbix agent is installed on supported operating systems to collect local metrics.

It can monitor:

* CPU utilization;
* memory usage;
* disk utilization;
* file systems;
* network interfaces;
* running processes;
* Windows services;
* Linux services;
* event logs;
* log files;
* installed software;
* open ports;
* custom scripts;
* application-specific metrics.

Two principal communication modes are available:

* passive checks;
* active checks.

The selected mode determines which component initiates the connection.

---

## Passive Agent Checks

During a passive check, the Zabbix server or Zabbix proxy initiates the connection to the agent.

```text
Zabbix Server or Proxy
          │
          │ TCP request
          ▼
     Zabbix Agent
          │
          │ Item value
          ▼
Zabbix Server or Proxy
```

The default Zabbix agent listening port is:

```text
TCP 10050
```

Passive checks require the monitored host to accept incoming connections from the Zabbix server or proxy.

Advantages include:

* simple item testing;
* immediate polling;
* direct control of polling intervals;
* straightforward troubleshooting.

Limitations include:

* inbound firewall access is required;
* NAT environments may complicate communication;
* large deployments create many server-initiated connections;
* remote endpoints may not be directly reachable.

Passive checks are suitable for trusted internal networks where the monitoring server can directly reach the monitored host.

---

## Active Agent Checks

During an active check, the Zabbix agent initiates the connection to the Zabbix server or proxy.

```text
Zabbix Agent
     │
     │ Requests active item list
     ▼
Zabbix Server or Proxy
     │
     │ Returns monitoring instructions
     ▼
Zabbix Agent
     │
     │ Sends collected values
     ▼
Zabbix Server or Proxy
```

The default Zabbix server trapper port is:

```text
TCP 10051
```

Advantages include:

* no inbound connection to the monitored host is required;
* better compatibility with NAT;
* useful for laptops and remote devices;
* reduced inbound firewall exposure;
* efficient log monitoring;
* suitable for distributed environments.

Limitations include:

* hostname configuration must match the Zabbix host definition;
* troubleshooting may be less intuitive;
* incorrect active server configuration can silently prevent data collection.

Active checks are recommended for devices located behind firewalls, NAT gateways or remote networks.

---

## Zabbix Agent and Zabbix Agent 2

Zabbix provides two agent implementations:

* Zabbix Agent;
* Zabbix Agent 2.

The classic Zabbix agent is lightweight and widely supported.

Zabbix Agent 2 provides a newer plugin-based architecture and expanded application monitoring capabilities.

Agent 2 is particularly useful for monitoring:

* Docker;
* PostgreSQL;
* MySQL;
* Redis;
* MongoDB;
* systemd;
* certificates;
* websites;
* hardware sensors;
* additional application services.

The exact set of available plugins depends on the installed version and operating system.

For new deployments, Zabbix Agent 2 is generally a strong default choice unless compatibility or operational requirements favor the classic agent.

---

## SNMP Monitoring

Simple Network Management Protocol is commonly used to monitor network and infrastructure devices.

Typical SNMP-monitored equipment includes:

* MikroTik routers;
* managed switches;
* firewalls;
* wireless controllers;
* access points;
* printers;
* UPS systems;
* storage systems;
* environmental sensors;
* power distribution units;
* server management controllers.

A typical SNMP monitoring flow is:

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

Common SNMP ports are:

| Protocol function |    Port |
| ----------------- | ------: |
| SNMP polling      | UDP 161 |
| SNMP traps        | UDP 162 |

SNMP versions commonly used are:

* SNMPv1;
* SNMPv2c;
* SNMPv3.

SNMPv3 should be preferred where supported because it provides authentication and encryption.

SNMPv1 and SNMPv2c use community strings and do not provide modern transport security.

---

## SNMP Polling

With SNMP polling, Zabbix periodically requests values from a monitored device.

Typical metrics include:

* interface status;
* interface traffic;
* packet errors;
* packet drops;
* CPU utilization;
* memory utilization;
* temperature;
* voltage;
* fan status;
* power supply state;
* device uptime.

Polling provides predictable and structured data collection.

However, polling intervals must be designed carefully to avoid excessive load on the Zabbix server, proxy, network or monitored device.

---

## SNMP Traps

SNMP traps are unsolicited event messages sent by monitored devices.

```text
Network Device
      │
      │ SNMP trap
      ▼
SNMP Trap Receiver
      │
      ▼
Zabbix Server
```

Traps can report events such as:

* interface failures;
* power supply failures;
* temperature alarms;
* authentication failures;
* routing changes;
* hardware faults;
* device restarts.

Traps should normally complement polling rather than replace it.

A missed trap may result in an undetected condition unless polling also verifies the current state.

---

## ICMP Monitoring

ICMP checks provide basic network availability and latency monitoring.

Typical ICMP measurements include:

* host reachability;
* packet loss;
* response time.

```text
Zabbix Server or Proxy
          │
          │ ICMP echo request
          ▼
      Monitored Host
          │
          │ ICMP echo reply
          ▼
Zabbix Server or Proxy
```

ICMP is useful for quickly determining whether a host is reachable.

However, a successful ping does not confirm that an application or service is functioning correctly.

Similarly, a failed ping does not always mean the device is offline because ICMP may be blocked by a firewall.

ICMP monitoring should therefore be combined with service-level checks.

---

## Simple Checks

Simple checks are performed directly by the Zabbix server or proxy without requiring an agent.

They can verify services such as:

* HTTP;
* HTTPS;
* SMTP;
* IMAP;
* POP3;
* FTP;
* SSH;
* LDAP;
* TCP ports.

Example:

```text
Zabbix Server
      │
      │ TCP connection test
      ▼
Application Server
      │
      ▼
Service availability result
```

Simple checks are useful for determining whether a service is listening and reachable.

They do not necessarily validate the complete application workflow.

---

## Web Monitoring

Zabbix web scenarios can simulate multi-step HTTP or HTTPS transactions.

A web scenario may verify:

* website availability;
* HTTP status codes;
* response time;
* page content;
* authentication steps;
* redirects;
* certificate-related access;
* multi-page workflows.

Example:

```text
Step 1: Open login page
Step 2: Submit credentials
Step 3: Verify dashboard content
Step 4: Confirm expected HTTP response
```

Web monitoring provides a more realistic service-level test than checking only whether TCP port 443 is open.

---

## External Checks

External checks execute scripts or binaries from the Zabbix server or proxy.

They can be used for:

* custom API queries;
* proprietary device monitoring;
* application health checks;
* certificate validation;
* integration with legacy systems;
* specialized network tests.

External checks should be designed carefully.

Poorly written scripts may:

* consume excessive CPU;
* block poller processes;
* create security risks;
* expose credentials;
* generate inconsistent output;
* cause monitoring delays.

Where possible, scripts should implement:

* timeouts;
* input validation;
* predictable output;
* secure credential handling;
* meaningful exit codes;
* structured logging.

---

## Zabbix Trapper Items

Trapper items allow external systems to send values to Zabbix.

Data may be submitted using:

```text
zabbix_sender
```

A typical flow is:

```text
Application or Script
          │
          │ Custom monitoring value
          ▼
   Zabbix Trapper
          │
          ▼
     Zabbix Server
```

Trapper items are useful for:

* backup results;
* application events;
* scheduled script output;
* business metrics;
* batch processing results;
* custom health checks;
* integration with automation platforms.

The Zabbix server or proxy receives trapper data on:

```text
TCP 10051
```

---

## IPMI Monitoring

Intelligent Platform Management Interface can be used to monitor compatible server hardware independently of the operating system.

Typical IPMI metrics include:

* temperature;
* fan speed;
* voltage;
* power state;
* chassis status;
* hardware sensors.

IPMI monitoring is useful for detecting hardware problems even if the operating system is unavailable.

Because IPMI interfaces are highly privileged management interfaces, they should be isolated on a dedicated management network.

---

## JMX Monitoring

Java Management Extensions monitoring is used for Java-based applications.

Examples include:

* Apache Tomcat;
* Java application servers;
* custom Java services;
* middleware platforms.

Zabbix uses the Java Gateway component for JMX monitoring.

```text
Zabbix Server or Proxy
          │
          ▼
     Java Gateway
          │
          ▼
     Java Application
          │
          ▼
       JMX Metrics
```

The Java Gateway translates monitoring requests between Zabbix and the JMX interface.

---

## Zabbix Proxy

A Zabbix proxy collects monitoring data on behalf of the central Zabbix server.

It is commonly deployed in:

* branch offices;
* remote data centers;
* customer networks;
* isolated network segments;
* high-latency environments;
* environments with unreliable WAN connections;
* managed service provider infrastructures.

A proxy can monitor hosts using:

* Zabbix agent;
* SNMP;
* ICMP;
* simple checks;
* external checks;
* IPMI;
* JMX through a Java Gateway;
* web scenarios.

The proxy buffers collected data locally and forwards it to the central server.

---

## Proxy Architecture

A distributed architecture may look like this:

```text
                         Central Data Center

                    ┌─────────────────────────┐
                    │      Zabbix Server      │
                    │                         │
                    │   Database + Frontend   │
                    └────────────┬────────────┘
                                 │
                          TCP 10051 / TLS
                                 │
             ┌───────────────────┼───────────────────┐
             │                   │                   │
             ▼                   ▼                   ▼
       Branch Proxy A      Branch Proxy B      Customer Proxy
             │                   │                   │
      ┌──────┴──────┐     ┌──────┴──────┐     ┌──────┴──────┐
      │             │     │             │     │             │
   Servers       Switches VMs         Routers Agents       SNMP
```

The central Zabbix server does not need direct network access to every monitored device when proxies are used.

Only communication between the proxy and server must be permitted.

---

## Active and Passive Proxies

Zabbix proxies can operate in active or passive mode.

### Active proxy

An active proxy initiates the connection to the Zabbix server.

```text
Zabbix Proxy
     │
     │ Outbound connection
     ▼
Zabbix Server
```

This mode is particularly useful when:

* the proxy is behind NAT;
* inbound access to the remote network is prohibited;
* customer networks must remain isolated;
* only outbound firewall connections are allowed.

### Passive proxy

In passive mode, the Zabbix server initiates the connection to the proxy.

```text
Zabbix Server
     │
     │ Connection to proxy
     ▼
Zabbix Proxy
```

This requires the central server to reach the proxy directly.

Active proxy mode is generally easier to deploy across NAT and restrictive firewall environments.

---

## Proxy Buffering

A Zabbix proxy stores collected values temporarily in its local database.

If communication with the central server is interrupted:

1. the proxy continues collecting data;
2. values are stored locally;
3. data is forwarded when connectivity is restored.

This provides resilience during:

* WAN outages;
* VPN failures;
* temporary routing issues;
* central server maintenance;
* firewall interruptions.

Proxy storage must be sized according to:

* number of monitored items;
* item update frequency;
* expected outage duration;
* value size;
* history synchronization rate.

A proxy is not a full replacement for the central Zabbix server.

It does not provide the complete frontend, global event processing or long-term centralized storage.

---

## Centralized Architecture

A centralized architecture uses one Zabbix server to communicate directly with all monitored hosts.

```text
                    Zabbix Server
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ▼                 ▼                 ▼
     Servers           Routers           Websites
```

This design is suitable for:

* small environments;
* single-site networks;
* laboratories;
* initial deployments;
* networks with reliable internal connectivity.

Advantages include:

* simple deployment;
* fewer components;
* easier administration;
* reduced infrastructure cost.

Limitations include:

* limited fault isolation;
* increased central server load;
* direct connectivity required;
* weak tolerance for WAN interruptions;
* reduced scalability across remote sites.

---

## Distributed Architecture

A distributed architecture uses one or more Zabbix proxies.

```text
                       Zabbix Server
                             │
             ┌───────────────┼───────────────┐
             │               │               │
             ▼               ▼               ▼
          Proxy A          Proxy B          Proxy C
             │               │               │
             ▼               ▼               ▼
          Site A           Site B           Site C
```

This design is suitable for:

* multi-site organizations;
* managed service providers;
* geographically distributed networks;
* remote customer environments;
* restricted security zones;
* unstable WAN links.

Advantages include:

* local data collection;
* reduced WAN polling traffic;
* buffering during outages;
* improved scalability;
* simplified firewall design;
* separation between customer environments.

Limitations include:

* additional infrastructure;
* additional databases;
* proxy lifecycle management;
* configuration complexity;
* more components to monitor.

---

## Reference Deployment Architecture

This guide uses a modular architecture that can be adapted to both small and enterprise environments.

```text
                              Internet
                                  │
                                  ▼
                        Firewall or VPN Gateway
                                  │
                                  ▼
                      Management / Monitoring VLAN
                                  │
          ┌───────────────────────┼───────────────────────┐
          │                       │                       │
          ▼                       ▼                       ▼
   Zabbix Server            Zabbix Database        Zabbix Frontend
          │
          │ TCP 10051
          │
     ┌────┴─────────────────────────────┐
     │                                  │
     ▼                                  ▼
Local Infrastructure               Remote Proxies
     │                                  │
 ┌───┼───────────┐                ┌─────┼─────────┐
 │   │           │                │     │         │
 ▼   ▼           ▼                ▼     ▼         ▼
VMs Routers   Applications      Branch Servers  Network Devices
```

The architecture separates the following functional layers:

1. presentation layer;
2. monitoring and processing layer;
3. database layer;
4. data collection layer;
5. monitored infrastructure layer;
6. notification and integration layer.

---

## Presentation Layer

The presentation layer consists primarily of the Zabbix frontend.

Its responsibilities include:

* displaying dashboards;
* visualizing metrics;
* showing problems;
* managing configuration;
* presenting reports;
* providing administrative access.

The presentation layer should be protected by:

* HTTPS;
* role-based access control;
* strong passwords;
* multi-factor authentication where available;
* restricted management network access;
* reverse proxy security policies;
* session timeout controls.

The frontend should not be exposed directly to the public Internet unless there is a documented business requirement and adequate protection.

---

## Processing Layer

The processing layer is provided by the Zabbix server.

It performs:

* poller operations;
* preprocessing;
* trigger evaluation;
* event generation;
* internal monitoring;
* discovery;
* escalation processing;
* proxy communication;
* alert execution.

Performance in this layer depends on:

* number of monitored hosts;
* number of enabled items;
* item update intervals;
* preprocessing complexity;
* trigger count;
* low-level discovery activity;
* number of proxies;
* database response time;
* worker process configuration.

The Zabbix server should monitor its own internal performance metrics.

---

## Database Layer

The database layer stores configuration and monitoring information.

For production deployments, this layer should provide:

* reliable storage;
* regular backups;
* sufficient IOPS;
* adequate memory;
* transaction consistency;
* monitored replication where required;
* retention management;
* tested recovery procedures.

The database should not be exposed to untrusted networks.

Only authorized components should be allowed to connect.

---

## Collection Layer

The collection layer includes:

* Zabbix agents;
* Zabbix proxies;
* SNMP polling;
* SNMP trap receivers;
* Java Gateway;
* external checks;
* web monitoring;
* IPMI pollers;
* custom integrations.

This layer interacts directly with monitored infrastructure.

Security controls should therefore restrict each collector to only the systems and protocols it requires.

---

## Notification and Integration Layer

Zabbix can send events to external systems.

Common integrations include:

* e-mail;
* SMS gateways;
* Microsoft Teams;
* Slack;
* Telegram;
* ticketing systems;
* incident management platforms;
* webhook endpoints;
* custom scripts.

A typical event flow is:

```text
Collected Metric
      │
      ▼
Trigger Evaluation
      │
      ▼
Problem Event
      │
      ▼
Action and Escalation
      │
      ▼
Notification or Ticket
```

Notification channels should be tested regularly.

A monitoring system that detects a failure but cannot notify the responsible team is operationally incomplete.

---

## Communication Matrix

The following table summarizes common Zabbix communication paths.

| Source                 | Destination            |      Default port | Protocol | Purpose                          |
| ---------------------- | ---------------------- | ----------------: | -------- | -------------------------------- |
| Zabbix Server or Proxy | Zabbix Agent           |             10050 | TCP      | Passive agent checks             |
| Zabbix Agent           | Zabbix Server or Proxy |             10051 | TCP      | Active checks                    |
| Zabbix Proxy           | Zabbix Server          |             10051 | TCP      | Proxy communication              |
| Zabbix Server          | Passive Zabbix Proxy   |             10051 | TCP      | Passive proxy polling            |
| Zabbix Server or Proxy | SNMP Device            |               161 | UDP      | SNMP polling                     |
| SNMP Device            | Trap Receiver          |               162 | UDP      | SNMP traps                       |
| Zabbix Server or Proxy | Network Host           |               N/A | ICMP     | Availability and latency         |
| Zabbix Frontend        | Database               | Database-specific | TCP      | Configuration and history access |
| Zabbix Server          | Database               | Database-specific | TCP      | Data storage and retrieval       |
| Zabbix Server or Proxy | Java Gateway           |             10052 | TCP      | JMX monitoring                   |
| Administrator          | Zabbix Frontend        |               443 | TCP      | Secure web access                |
| Zabbix Server          | SMTP Server            |    25, 465 or 587 | TCP      | E-mail notifications             |

Ports may differ depending on the environment.

Firewall rules should be based on actual communication requirements rather than broad network access.

---

## Firewall Design

A monitoring architecture requires carefully planned firewall policies.

The objective is not to allow unrestricted communication between the Zabbix server and all monitored networks.

Instead, allow only the required source, destination, protocol and port combinations.

Example policy:

```text
Zabbix Server → Linux Servers       TCP 10050
Zabbix Server → Windows Servers     TCP 10050
Zabbix Server → Network Devices     UDP 161
Zabbix Agents → Zabbix Server       TCP 10051
Zabbix Proxies → Zabbix Server      TCP 10051
Administrators → Zabbix Frontend    TCP 443
```

Recommended firewall principles include:

* deny by default;
* use explicit source addresses;
* use explicit destination addresses;
* allow only required ports;
* separate management and production traffic;
* log denied monitoring traffic where useful;
* review obsolete rules;
* avoid exposing agents to the Internet;
* encrypt untrusted communication paths.

---

## Network Segmentation

Monitoring systems frequently require access to many infrastructure components.

This makes segmentation particularly important.

A recommended design uses a dedicated monitoring or management network.

```text
                         Core Firewall
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
          ▼                   ▼                   ▼
     User Network        Server Network     Monitoring VLAN
                                                    │
                                                    ▼
                                              Zabbix Server
```

Benefits include:

* reduced attack surface;
* easier firewall management;
* better traffic visibility;
* isolation from user networks;
* improved auditability;
* simpler access control.

The monitoring network should not automatically receive unrestricted access to every VLAN.

Access should be granted according to the monitoring methods in use.

---

## Monitoring Across VPN Connections

Remote networks may be monitored across site-to-site VPN connections.

```text
Central Zabbix Server
          │
          │ Encrypted VPN Tunnel
          ▼
      Remote Network
          │
     ┌────┼────────┐
     │    │        │
     ▼    ▼        ▼
  Server Router  Switch
```

This architecture may be suitable when:

* the number of remote hosts is small;
* the VPN is stable;
* latency is low;
* direct polling is operationally acceptable.

For larger or less reliable remote environments, a local Zabbix proxy is preferable.

```text
Central Zabbix Server
          │
          │ Encrypted connection
          ▼
   Remote Zabbix Proxy
          │
          ▼
 Remote Infrastructure
```

The proxy reduces the number of monitoring connections crossing the VPN and buffers data during tunnel outages.

---

## Monitoring Across the Internet

Direct monitoring across the public Internet should be avoided where possible.

Agents, SNMP services and management interfaces should not be exposed publicly without strong justification.

Preferred approaches include:

* site-to-site VPN;
* remote-access VPN;
* active Zabbix proxy;
* active Zabbix agent;
* encrypted TLS communication;
* private management networks.

A secure Internet-based proxy design may look like this:

```text
Remote Customer Network
          │
          ▼
   Active Zabbix Proxy
          │
          │ Outbound TLS connection
          ▼
        Internet
          │
          ▼
Central Zabbix Server
```

Only the proxy initiates communication.

The central server does not require direct access to monitored customer devices.

---

## Encryption

Zabbix communication can be protected using TLS.

Supported trust models may include:

* unencrypted communication;
* pre-shared key encryption;
* certificate-based encryption.

Unencrypted communication should be limited to trusted and isolated networks.

Pre-shared keys provide a practical encryption method for many environments.

Certificate-based encryption is appropriate where:

* certificate lifecycle management already exists;
* stronger identity management is required;
* large-scale automation is available;
* compliance requirements mandate certificates.

Encryption should be considered for:

* agent communication;
* proxy communication;
* server communication;
* frontend access;
* database connections where required;
* communication across untrusted networks.

---

## High Availability

Monitoring becomes increasingly important as infrastructure complexity grows.

The monitoring platform itself should therefore avoid obvious single points of failure.

Potential failure points include:

* Zabbix server;
* database server;
* frontend web server;
* storage;
* network connectivity;
* proxy server;
* notification service;
* DNS;
* certificate infrastructure.

High availability design may include:

* Zabbix server high availability nodes;
* database replication;
* database clustering;
* redundant web frontends;
* redundant load balancers;
* redundant storage;
* multiple notification channels;
* monitored backup systems.

High availability does not remove the need for backups.

Replication protects availability, while backups protect recoverability.

---

## Basic High Availability Design

A simplified high availability architecture may look like this:

```text
                        Load Balancer
                             │
               ┌─────────────┴─────────────┐
               │                           │
               ▼                           ▼
        Zabbix Frontend 1            Zabbix Frontend 2
               │                           │
               └─────────────┬─────────────┘
                             │
                             ▼
                  Zabbix Server HA Cluster
                     ┌───────┴───────┐
                     │               │
                     ▼               ▼
                  Node 1           Node 2
                             │
                             ▼
                    Highly Available Database
```

The precise implementation depends on:

* infrastructure size;
* acceptable downtime;
* recovery objectives;
* database technology;
* operational expertise;
* available budget.

---

## Capacity Planning

Capacity planning should be performed before production deployment.

Important variables include:

* number of monitored hosts;
* number of items per host;
* item update intervals;
* values collected per second;
* history retention period;
* trend retention period;
* event volume;
* log monitoring volume;
* number of proxies;
* number of users;
* dashboard complexity;
* database storage growth;
* housekeeping strategy.

A simplified estimate of collected values can be calculated as:

```text
Values per second =
Total enabled items / Average update interval
```

Example:

```text
10,000 enabled items / 60 seconds = approximately 167 values per second
```

This is only a simplified estimate.

Real workloads are affected by:

* unsupported items;
* trapper values;
* dependent items;
* discovery;
* preprocessing;
* burst traffic;
* agent availability;
* proxy synchronization;
* log item activity.

---

## Polling Interval Design

Not every metric requires the same update interval.

Example classification:

| Metric type                   | Example interval |
| ----------------------------- | ---------------: |
| Critical service availability |    30–60 seconds |
| CPU and memory utilization    |       60 seconds |
| Interface traffic             |       60 seconds |
| Disk capacity                 |        5 minutes |
| Installed software inventory  |       1–24 hours |
| Configuration inventory       |       1–24 hours |
| Certificate expiration        |       1–12 hours |
| Static asset information      |         24 hours |

Very short intervals increase:

* server load;
* proxy load;
* network traffic;
* database writes;
* storage requirements.

Intervals should match the operational value of the data.

Collecting a static serial number every 30 seconds provides no practical benefit.

---

## History and Trend Data

Zabbix stores two principal forms of time-series data:

* history;
* trends.

History contains detailed collected values.

Trend data contains aggregated information for longer-term analysis.

Retention planning should consider:

* troubleshooting requirements;
* reporting periods;
* compliance requirements;
* database capacity;
* performance impact.

Example policy:

| Data type        |                     Example retention |
| ---------------- | ------------------------------------: |
| Detailed history |                             7–30 days |
| Trend data       |                      365 days or more |
| Events           | According to operational requirements |
| Audit records    |    According to security requirements |

Long retention periods should be based on a clear business requirement.

---

## Housekeeping

Housekeeping removes expired data from the Zabbix database.

It may process:

* history;
* trends;
* events;
* alerts;
* discovery data;
* internal records;
* audit information.

In large environments, default housekeeping may create significant database load.

Alternative strategies may include:

* database partitioning;
* TimescaleDB retention policies;
* scheduled database maintenance;
* carefully tuned retention periods.

Housekeeping performance should be monitored.

An overloaded housekeeping process can cause continuous database pressure.

---

## Database Storage Design

Database storage should be sized according to expected write volume and retention.

Important factors include:

* values per second;
* data type;
* history retention;
* trend retention;
* event volume;
* indexing overhead;
* database engine;
* compression;
* partitioning;
* backup requirements.

Production recommendations include:

* use SSD or NVMe storage;
* avoid slow shared storage unless validated;
* monitor database latency;
* monitor free space;
* reserve growth capacity;
* separate database backups from production storage;
* test restore procedures;
* document retention assumptions.

Storage exhaustion can stop data collection and may damage database availability.

---

## Proxy Placement Strategy

A proxy should be placed close to the infrastructure it monitors.

Good proxy placement examples include:

* one proxy per branch office;
* one proxy per customer;
* one proxy per data center;
* one proxy per security zone;
* one proxy per isolated industrial network;
* one proxy per large cloud environment.

Avoid placing a single proxy across unrelated trust zones unless the design has been reviewed carefully.

Proxy placement should consider:

* network latency;
* firewall boundaries;
* customer isolation;
* item volume;
* database capacity;
* failure domains;
* maintenance responsibility.

---

## Managed Service Provider Architecture

Managed service providers frequently monitor multiple independent customer environments.

A recommended design uses a dedicated proxy for each customer or customer site.

```text
                         MSP Zabbix Server
                                │
             ┌──────────────────┼──────────────────┐
             │                  │                  │
             ▼                  ▼                  ▼
       Customer A Proxy   Customer B Proxy   Customer C Proxy
             │                  │                  │
             ▼                  ▼                  ▼
       Customer A LAN     Customer B LAN     Customer C LAN
```

This design provides:

* logical separation;
* simplified firewall rules;
* independent buffering;
* easier troubleshooting;
* reduced cross-customer access;
* clearer ownership;
* easier customer offboarding.

Customer host groups, user permissions and naming conventions should also be separated consistently.

---

## Naming Conventions

Consistent naming is an architectural requirement.

Recommended host naming patterns may include:

```text
SITE-ROLE-NUMBER
```

Examples:

```text
BUD-DC-01
BUD-HV-01
BUD-SW-01
GYR-FW-01
CUSTOMERA-SRV-01
```

Useful naming elements include:

* customer;
* physical site;
* device role;
* environment;
* sequence number.

Avoid names such as:

```text
Server1
NewRouter
TestMachine
PC-John
```

Names should remain meaningful even when viewed outside the original network context.

---

## Host Groups

Host groups should represent operational or administrative boundaries.

Possible host group structures include:

```text
Customers/Customer-A/Servers
Customers/Customer-A/Network
Customers/Customer-A/Applications

Sites/Budapest
Sites/Győr
Sites/Debrecen

Infrastructure/Linux
Infrastructure/Windows
Infrastructure/Network
Infrastructure/Virtualization
```

Host groups are used for:

* permissions;
* filtering;
* actions;
* maintenance;
* dashboards;
* reporting;
* inventory organization.

A host may belong to multiple groups.

---

## Template Architecture

Templates provide reusable monitoring configuration.

A layered template model is recommended.

Example:

```text
Template OS Linux
        │
        ├── Template App SSH Service
        ├── Template App Nginx
        └── Template App PostgreSQL
```

A host can receive multiple templates according to its role.

Example:

```text
Host: WEB-SRV-01

Assigned templates:
- Linux by Zabbix agent
- Nginx by Zabbix agent
- Website certificate monitoring
- Custom backup monitoring
```

Avoid creating one oversized template containing every possible item.

Modular templates are easier to:

* maintain;
* test;
* reuse;
* update;
* troubleshoot;
* assign consistently.

---

## Dependent Items

Dependent items reduce duplicate data collection.

A master item collects a larger dataset, such as JSON output.

Dependent items then extract individual values.

```text
Master Item
    │
    │ JSON payload
    ▼
Preprocessing
    │
    ├── CPU value
    ├── Memory value
    ├── Service state
    └── Connection count
```

Benefits include:

* fewer network requests;
* reduced API load;
* lower device load;
* consistent data collection;
* easier bulk metric processing.

Dependent items are particularly effective for:

* REST APIs;
* JSON endpoints;
* application status pages;
* cloud APIs;
* container metrics;
* custom scripts.

---

## Low-Level Discovery

Low-level discovery automatically identifies entities such as:

* file systems;
* network interfaces;
* mounted disks;
* services;
* processes;
* SNMP interfaces;
* databases;
* containers;
* virtual machines;
* certificates.

Discovery creates items, triggers and graphs from prototypes.

This improves scalability but must be controlled carefully.

Poorly designed discovery can create:

* thousands of unnecessary items;
* excessive database growth;
* obsolete entities;
* duplicate triggers;
* excessive polling load.

Discovery filters and override rules should be designed before large-scale deployment.

---

## Monitoring the Monitoring System

The Zabbix platform must monitor itself.

Important internal metrics include:

* Zabbix server availability;
* values processed per second;
* queue size;
* unsupported items;
* poller utilization;
* unreachable poller utilization;
* preprocessing utilization;
* history syncer utilization;
* cache utilization;
* database latency;
* proxy last access;
* proxy queue;
* frontend availability;
* database free space;
* notification failures;
* backup status;
* certificate expiration.

A self-monitoring failure should generate an alert through an independent or redundant notification path where possible.

---

## Failure Scenarios

Architecture planning should consider likely failure scenarios.

### Zabbix server failure

Effects:

* polling stops;
* trigger processing stops;
* notifications stop;
* proxy data remains buffered temporarily.

### Database failure

Effects:

* server processing is severely affected;
* frontend becomes unavailable or unusable;
* new data cannot be stored;
* configuration cannot be retrieved reliably.

### Proxy failure

Effects:

* hosts assigned to the proxy stop being monitored;
* no new values are collected;
* central server remains operational.

### WAN failure

Effects:

* direct remote monitoring fails;
* active proxies buffer data;
* central monitoring may show communication problems.

### Notification service failure

Effects:

* problems are detected;
* alerts may not reach administrators.

Every critical component should have:

* monitoring;
* backups;
* recovery documentation;
* escalation procedures;
* defined ownership.

---

## Security Boundaries

The monitoring platform crosses multiple trust boundaries.

It may collect data from:

* domain controllers;
* database servers;
* firewalls;
* backup systems;
* hypervisors;
* customer networks;
* management interfaces.

Compromise of the monitoring system may expose sensitive infrastructure information.

Security controls should include:

* dedicated service accounts;
* least privilege;
* encrypted communication;
* network segmentation;
* restricted frontend access;
* secret protection;
* audit logging;
* regular updates;
* template review;
* script review;
* controlled API access;
* secure backups.

Credentials should not be stored directly in item keys, scripts or public repositories.

Use macros, secret stores or controlled configuration mechanisms where appropriate.

---

## Architecture Selection Guide

The following table provides a basic architecture selection guide.

| Environment                     | Recommended architecture                                       |
| ------------------------------- | -------------------------------------------------------------- |
| Small single-site office        | Centralized Zabbix server                                      |
| Medium single-site organization | Separate server and database where required                    |
| Multiple branch offices         | Central server with one proxy per branch                       |
| Managed service provider        | Central server with customer-specific proxies                  |
| Isolated security zones         | Proxy per security zone                                        |
| Unreliable WAN connection       | Active proxy with sufficient buffer                            |
| Mobile or NAT-based endpoints   | Active agent checks                                            |
| Large enterprise                | Distributed proxies with database and server capacity planning |
| Critical infrastructure         | High availability server and database design                   |

The simplest architecture that satisfies availability, security and scalability requirements is usually the most maintainable.

---

## Recommended Baseline Architecture

For a production-oriented deployment, the following baseline is recommended:

```text
Central Zabbix Environment
├── Zabbix Server
├── PostgreSQL Database
├── Zabbix Frontend
├── HTTPS Reverse Proxy
├── Regular Database Backup
└── External Notification Channel

Remote Site
├── Active Zabbix Proxy
├── Local Proxy Database
├── Zabbix Agents
├── SNMP Devices
└── Outbound TLS Connection to Central Server
```

Core principles:

* use a dedicated monitoring network where possible;
* use active proxies for remote sites;
* use TLS across untrusted networks;
* restrict firewall access;
* use modular templates;
* monitor Zabbix internally;
* size database storage before deployment;
* define retention policies;
* back up configuration and database data;
* test notification and recovery procedures.

---

## Architecture Validation Checklist

Before deploying the monitoring platform, verify the following.

### Core components

* [ ] Zabbix server role defined
* [ ] Database platform selected
* [ ] Frontend web server selected
* [ ] Storage requirements estimated
* [ ] Backup strategy documented
* [ ] Time synchronization configured

### Network design

* [ ] Monitoring network identified
* [ ] Required communication paths documented
* [ ] Firewall ports defined
* [ ] Remote sites identified
* [ ] VPN or proxy strategy selected
* [ ] DNS resolution verified
* [ ] Routing verified

### Security

* [ ] HTTPS enabled
* [ ] Agent communication protected
* [ ] Proxy communication protected
* [ ] SNMPv3 used where supported
* [ ] Service accounts follow least privilege
* [ ] Credentials stored securely
* [ ] Management access restricted
* [ ] Audit requirements documented

### Scalability

* [ ] Host count estimated
* [ ] Item count estimated
* [ ] Values per second estimated
* [ ] Polling intervals reviewed
* [ ] History retention defined
* [ ] Trend retention defined
* [ ] Proxy capacity reviewed
* [ ] Database growth estimated

### Availability

* [ ] Server recovery procedure documented
* [ ] Database recovery procedure documented
* [ ] Proxy outage behavior understood
* [ ] Notification redundancy considered
* [ ] Monitoring platform self-monitoring enabled
* [ ] Backup restoration tested

---

## Common Architecture Mistakes

Avoid the following design errors:

* placing every Zabbix component on an undersized server;
* exposing Zabbix agents directly to the Internet;
* using unrestricted firewall rules;
* monitoring remote sites directly across unstable WAN links;
* using SNMPv2c where SNMPv3 is available;
* collecting every metric at extremely short intervals;
* retaining detailed history indefinitely;
* failing to monitor database growth;
* using one oversized template for every host;
* deploying proxies without sufficient local storage;
* storing credentials in plain text scripts;
* failing to monitor the Zabbix platform itself;
* relying on only one notification channel;
* operating without tested backups;
* using inconsistent host naming conventions.

A monitoring platform should reduce operational risk rather than introduce additional hidden dependencies.

---

## Summary

The monitoring architecture defines how Zabbix components communicate with the monitored infrastructure and with each other.

A robust architecture should provide:

* centralized visibility;
* distributed data collection;
* secure communication;
* predictable performance;
* fault isolation;
* scalable templates;
* controlled retention;
* reliable notifications;
* tested recovery procedures.

Small environments can begin with a centralized Zabbix server.

As the monitored infrastructure grows, Zabbix proxies, separated database services, high availability components and stronger security boundaries can be introduced without redesigning the entire monitoring model.

The next chapter covers the installation and initial deployment of the Zabbix platform.

---

## Next Chapter

Continue with:

[Next: Installation →](03-installation.md)
