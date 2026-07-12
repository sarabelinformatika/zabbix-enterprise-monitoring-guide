# Installation

This chapter describes the installation of a production-ready Zabbix monitoring environment.

The reference architecture uses:

* Debian Linux;
* PostgreSQL;
* Nginx;
* PHP-FPM;
* Zabbix Server;
* Zabbix Frontend;
* Zabbix Agent 2.

The exact package names and repository paths may differ between operating system and Zabbix versions.

Before installation, verify the currently supported operating systems, database platforms and package repositories for the selected Zabbix release.

---

## Installation Architecture

The basic deployment described in this chapter uses a single Linux server.

```text
                    Administrator
                          │
                          ▼
                    Web Browser
                          │
                       HTTPS
                          │
                          ▼
                   Nginx Frontend
                          │
                          ▼
                   Zabbix Frontend
                          │
                          ▼
                    PostgreSQL
                          ▲
                          │
                    Zabbix Server
                          │
                          ▼
                  Monitored Systems
```

For smaller environments, the following components may run on the same virtual machine:

* Zabbix Server;
* PostgreSQL;
* Nginx;
* PHP-FPM;
* Zabbix Frontend;
* Zabbix Agent 2.

For larger deployments, the database and frontend may be separated.

---

## System Requirements

The required resources depend on:

* number of monitored hosts;
* number of monitored items;
* item update intervals;
* history retention;
* trend retention;
* database workload;
* number of simultaneous users;
* dashboard complexity;
* proxy synchronization volume.

A small deployment may start with:

| Resource |                    Recommended minimum |
| -------- | -------------------------------------: |
| CPU      |                                 2 vCPU |
| Memory   |                               4 GB RAM |
| Storage  |                              40 GB SSD |
| Network  | 1 Gbit/s virtual or physical interface |

A more practical production baseline is:

| Resource |                       Recommended baseline |
| -------- | -----------------------------------------: |
| CPU      |                                     4 vCPU |
| Memory   |                                   8 GB RAM |
| Storage  |                              80–150 GB SSD |
| Network  |                                   1 Gbit/s |
| Backup   | External database and configuration backup |

Storage should be increased according to history retention and collected values per second.

---

## Recommended Server Layout

Example server parameters:

```text
Hostname: ZBX-SRV-01
Operating system: Debian
IP address: 192.168.10.20
Subnet mask: 255.255.255.0
Default gateway: 192.168.10.1
DNS server: 192.168.10.10
Timezone: Europe/Budapest
```

Recommended DNS records:

```text
zabbix.example.local
monitoring.example.local
```

For public or remote administrative access, use a valid domain name and trusted TLS certificate.

---

## Pre-Installation Checklist

Before installing Zabbix, verify the following:

* [ ] Static IP address configured
* [ ] Hostname configured
* [ ] DNS resolution working
* [ ] Default gateway reachable
* [ ] Internet access available for package installation
* [ ] Time synchronization configured
* [ ] Correct timezone configured
* [ ] Required firewall rules documented
* [ ] Database credentials prepared
* [ ] Backup destination available
* [ ] HTTPS hostname selected

---

## Update the Operating System

Update the installed packages before adding the Zabbix repository.

```bash
apt update
apt full-upgrade -y
```

Remove packages that are no longer required:

```bash
apt autoremove -y
```

Verify the operating system version:

```bash
cat /etc/os-release
```

Verify the kernel:

```bash
uname -a
```

A reboot may be required after kernel or system library updates.

```bash
reboot
```

---

## Configure the Hostname

Set the server hostname:

```bash
hostnamectl set-hostname ZBX-SRV-01
```

Verify it:

```bash
hostnamectl
```

Update `/etc/hosts`:

```bash
nano /etc/hosts
```

Example:

```text
127.0.0.1       localhost
127.0.1.1       ZBX-SRV-01
192.168.10.20   ZBX-SRV-01 zabbix.example.local
```

Test local resolution:

```bash
getent hosts ZBX-SRV-01
getent hosts zabbix.example.local
```

---

## Configure the Timezone

Set the correct timezone:

```bash
timedatectl set-timezone Europe/Budapest
```

Verify time configuration:

```bash
timedatectl
```

Expected output should confirm:

```text
Time zone: Europe/Budapest
System clock synchronized: yes
NTP service: active
```

Accurate time synchronization is critical for:

* event correlation;
* trigger evaluation;
* log monitoring;
* proxy synchronization;
* audit records;
* certificate validation.

---

## Configure Time Synchronization

Install Chrony:

```bash
apt install chrony -y
```

Enable and start the service:

```bash
systemctl enable --now chrony
```

Verify synchronization:

```bash
chronyc tracking
```

Display configured time sources:

```bash
chronyc sources -v
```

---

## Configure a Static IP Address

The exact configuration method depends on the Debian installation and network stack.

Possible implementations include:

* `/etc/network/interfaces`;
* systemd-networkd;
* NetworkManager;
* cloud-init.

Example using `/etc/network/interfaces`:

```bash
nano /etc/network/interfaces
```

Example configuration:

```text
auto ens18
iface ens18 inet static
    address 192.168.10.20/24
    gateway 192.168.10.1
    dns-nameservers 192.168.10.10 1.1.1.1
```

Restart networking carefully:

```bash
systemctl restart networking
```

When connected remotely, changing network configuration may terminate the SSH session.

Verify:

```bash
ip address show
ip route show
resolvectl status
```

Test connectivity:

```bash
ping -c 4 192.168.10.1
ping -c 4 1.1.1.1
ping -c 4 deb.debian.org
```

---

## Install Required Utilities

Install common administrative and troubleshooting packages:

```bash
apt install -y \
    curl \
    wget \
    gnupg \
    ca-certificates \
    lsb-release \
    nano \
    vim \
    unzip \
    tar \
    jq \
    net-tools \
    iproute2 \
    dnsutils \
    traceroute \
    tcpdump \
    htop \
    sudo
```

These tools are useful for:

* repository configuration;
* network testing;
* API testing;
* log analysis;
* troubleshooting;
* package downloads.

---

## Configure the Firewall

A host-based firewall should permit only required connections.

Typical inbound ports include:

| Port  | Protocol | Purpose                                 |
| ----- | -------- | --------------------------------------- |
| 22    | TCP      | SSH administration                      |
| 80    | TCP      | HTTP redirect or certificate validation |
| 443   | TCP      | Zabbix frontend                         |
| 10050 | TCP      | Local Zabbix agent passive checks       |
| 10051 | TCP      | Zabbix server and proxy communication   |

The exact rules depend on the architecture.

Install UFW if required:

```bash
apt install ufw -y
```

Set default policies:

```bash
ufw default deny incoming
ufw default allow outgoing
```

Allow SSH from the management network:

```bash
ufw allow from 192.168.10.0/24 to any port 22 proto tcp
```

Allow HTTPS:

```bash
ufw allow 443/tcp
```

Allow HTTP if required:

```bash
ufw allow 80/tcp
```

Allow Zabbix server communication from authorized networks:

```bash
ufw allow from 192.168.10.0/24 to any port 10051 proto tcp
```

Enable the firewall:

```bash
ufw enable
```

Verify the rules:

```bash
ufw status verbose
```

Do not enable firewall rules without confirming that SSH access will remain available.

---

## Install the Zabbix Repository

Download the repository package for the selected operating system and Zabbix release.

Example structure:

```bash
wget https://repo.zabbix.com/zabbix/<VERSION>/release/debian/pool/main/z/zabbix-release/zabbix-release_latest_<VERSION>+debian<DEBIAN_VERSION>_all.deb
```

Install the repository package:

```bash
dpkg -i zabbix-release_latest_*.deb
```

Update package metadata:

```bash
apt update
```

Verify that Zabbix packages are available:

```bash
apt-cache policy zabbix-server-pgsql
```

The repository URL must match:

* the installed Debian version;
* the selected Zabbix release;
* the required architecture.

---

## Install PostgreSQL

Install PostgreSQL:

```bash
apt install postgresql postgresql-contrib -y
```

Enable and start PostgreSQL:

```bash
systemctl enable --now postgresql
```

Verify service status:

```bash
systemctl status postgresql
```

Check the installed version:

```bash
psql --version
```

List PostgreSQL clusters:

```bash
pg_lsclusters
```

---

## Create the Zabbix Database User

Switch to the PostgreSQL administrative user:

```bash
sudo -u postgres psql
```

Create the Zabbix database user:

```sql
CREATE USER zabbix WITH PASSWORD 'CHANGE_THIS_STRONG_PASSWORD';
```

Create the database:

```sql
CREATE DATABASE zabbix OWNER zabbix;
```

Exit PostgreSQL:

```sql
\q
```

Use a strong, unique password.

Do not reuse:

* root passwords;
* domain administrator passwords;
* frontend administrator passwords;
* passwords used on monitored devices.

---

## Verify Database Access

Test authentication:

```bash
psql -h 127.0.0.1 -U zabbix -d zabbix
```

Enter the database password when prompted.

Verify the current connection:

```sql
\conninfo
```

Exit:

```sql
\q
```

If authentication fails, review:

```text
/etc/postgresql/<VERSION>/main/pg_hba.conf
```

Common local authentication methods include:

```text
scram-sha-256
md5
peer
```

After modifying PostgreSQL configuration:

```bash
systemctl restart postgresql
```

---

## Install Zabbix Server and Frontend Packages

Install the required Zabbix components:

```bash
apt install -y \
    zabbix-server-pgsql \
    zabbix-frontend-php \
    php8.2-pgsql \
    zabbix-nginx-conf \
    zabbix-sql-scripts \
    zabbix-agent2
```

Package names may differ depending on:

* Debian version;
* PHP version;
* Zabbix version;
* repository structure.

Verify installed packages:

```bash
dpkg -l | grep zabbix
```

---

## Import the Initial Database Schema

Locate the PostgreSQL schema file:

```bash
find /usr/share/zabbix-sql-scripts -type f | grep server.sql
```

The file may be compressed.

Example import command:

```bash
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz \
    | sudo -u zabbix psql zabbix
```

Alternatively:

```bash
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz \
    | psql -h 127.0.0.1 -U zabbix -d zabbix
```

The import may take several minutes depending on system performance.

Verify database tables:

```bash
sudo -u postgres psql -d zabbix -c "\dt"
```

Verify the table count:

```bash
sudo -u postgres psql -d zabbix -tAc \
"SELECT count(*) FROM information_schema.tables WHERE table_schema='public';"
```

A successful import creates numerous Zabbix tables.

---

## Configure the Zabbix Server

Edit the server configuration:

```bash
nano /etc/zabbix/zabbix_server.conf
```

Configure the database settings:

```text
DBHost=127.0.0.1
DBName=zabbix
DBUser=zabbix
DBPassword=CHANGE_THIS_STRONG_PASSWORD
```

Recommended initial settings may include:

```text
LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=0
PidFile=/run/zabbix/zabbix_server.pid
SocketDir=/run/zabbix
```

Do not change performance-related process counts before establishing a baseline.

Examples of settings that may require later tuning:

```text
StartPollers=
StartPreprocessors=
StartTrappers=
StartPingers=
StartDiscoverers=
CacheSize=
HistoryCacheSize=
HistoryIndexCacheSize=
TrendCacheSize=
ValueCacheSize=
Timeout=
```

Premature tuning can make troubleshooting more difficult.

---

## Protect Database Credentials

Restrict access to the Zabbix server configuration:

```bash
chown root:zabbix /etc/zabbix/zabbix_server.conf
chmod 640 /etc/zabbix/zabbix_server.conf
```

Verify:

```bash
ls -l /etc/zabbix/zabbix_server.conf
```

Expected permissions:

```text
-rw-r----- root zabbix
```

The configuration file contains sensitive database credentials and must not be world-readable.

---

## Configure Nginx

Edit the Zabbix Nginx configuration:

```bash
nano /etc/zabbix/nginx.conf
```

Configure the listening port and hostname:

```nginx
listen 80;
server_name zabbix.example.local;
```

Example configuration:

```nginx
server {
    listen 80;
    server_name zabbix.example.local;

    root /usr/share/zabbix;

    index index.php;

    location = /favicon.ico {
        log_not_found off;
    }

    location / {
        try_files $uri $uri/ =404;
    }

    location /assets {
        access_log off;
        expires 10d;
    }

    location ~ /\.ht {
        deny all;
    }

    location ~ /(api\/|conf[^\.]|include|locale) {
        deny all;
        return 404;
    }

    location /vendor {
        deny all;
        return 404;
    }

    location ~ [^/]\.php(/|$) {
        fastcgi_pass unix:/run/php/zabbix.sock;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_index index.php;

        fastcgi_param DOCUMENT_ROOT /usr/share/zabbix;
        fastcgi_param SCRIPT_FILENAME /usr/share/zabbix$fastcgi_script_name;
        fastcgi_param PATH_TRANSLATED /usr/share/zabbix$fastcgi_script_name;

        include fastcgi_params;
        fastcgi_param QUERY_STRING $query_string;
        fastcgi_param REQUEST_METHOD $request_method;
        fastcgi_param CONTENT_TYPE $content_type;
        fastcgi_param CONTENT_LENGTH $content_length;

        fastcgi_intercept_errors on;
        fastcgi_ignore_client_abort off;
        fastcgi_connect_timeout 60;
        fastcgi_send_timeout 180;
        fastcgi_read_timeout 180;
        fastcgi_buffer_size 128k;
        fastcgi_buffers 4 256k;
        fastcgi_busy_buffers_size 256k;
    }
}
```

Test the Nginx configuration:

```bash
nginx -t
```

---

## Configure PHP-FPM

Locate the Zabbix PHP-FPM configuration:

```bash
ls -l /etc/zabbix/php-fpm.conf
```

Edit it:

```bash
nano /etc/zabbix/php-fpm.conf
```

Set the timezone:

```text
php_value[date.timezone] = Europe/Budapest
```

Recommended PHP values may include:

```text
php_value[max_execution_time] = 300
php_value[memory_limit] = 256M
php_value[post_max_size] = 32M
php_value[upload_max_filesize] = 16M
php_value[max_input_time] = 300
php_value[session.auto_start] = 0
php_value[mbstring.func_overload] = 0
php_value[always_populate_raw_post_data] = -1
```

Restart PHP-FPM after changes.

The exact service name depends on the installed PHP version.

Example:

```bash
systemctl restart php8.2-fpm
```

---

## Start Zabbix Services

Enable and start the required services:

```bash
systemctl enable --now \
    zabbix-server \
    zabbix-agent2 \
    nginx \
    php8.2-fpm
```

Restart all components:

```bash
systemctl restart \
    zabbix-server \
    zabbix-agent2 \
    nginx \
    php8.2-fpm
```

Check service status:

```bash
systemctl status zabbix-server
systemctl status zabbix-agent2
systemctl status nginx
systemctl status php8.2-fpm
```

---

## Verify Listening Ports

Check active listening sockets:

```bash
ss -tulpn
```

Expected ports may include:

```text
TCP 80       Nginx
TCP 443      Nginx after HTTPS configuration
TCP 10050    Zabbix Agent 2
TCP 10051    Zabbix Server
TCP 5432     PostgreSQL, usually local only
TCP 22       SSH
```

Filter Zabbix ports:

```bash
ss -lntp | grep -E '10050|10051'
```

Verify Nginx:

```bash
ss -lntp | grep -E ':80|:443'
```

---

## Review Zabbix Server Logs

Display the latest server log entries:

```bash
tail -n 100 /var/log/zabbix/zabbix_server.log
```

Follow the log in real time:

```bash
tail -f /var/log/zabbix/zabbix_server.log
```

Using journalctl:

```bash
journalctl -u zabbix-server -n 100 --no-pager
```

Typical errors include:

* database authentication failure;
* missing database schema;
* incorrect database password;
* permission problems;
* unsupported database version;
* unavailable database server;
* invalid configuration parameter;
* port already in use.

---

## Review Nginx and PHP Logs

Check Nginx errors:

```bash
tail -n 100 /var/log/nginx/error.log
```

Check access logs:

```bash
tail -n 100 /var/log/nginx/access.log
```

Check PHP-FPM:

```bash
journalctl -u php8.2-fpm -n 100 --no-pager
```

Common frontend problems include:

* PHP socket mismatch;
* invalid Nginx configuration;
* missing PHP extension;
* incorrect filesystem permissions;
* unavailable database;
* incorrect frontend database settings.

---

## Complete the Web Installation

Open the Zabbix frontend:

```text
http://zabbix.example.local
```

The installation wizard verifies:

* PHP version;
* PHP extensions;
* PHP memory limit;
* PHP execution limits;
* database connectivity;
* frontend configuration.

During database configuration, enter:

```text
Database type: PostgreSQL
Database host: 127.0.0.1
Database port: 5432
Database name: zabbix
Database user: zabbix
Database password: configured database password
```

Set the Zabbix server name:

```text
SARABEL Monitoring
```

Complete the wizard.

---

## Default Frontend Login

The default Zabbix administrative account is commonly:

```text
Username: Admin
Password: zabbix
```

Usernames may be case-sensitive.

Immediately after the first login:

1. change the default administrator password;
2. create a named administrative user;
3. avoid using the default account for normal administration;
4. review frontend permissions;
5. configure authentication security.

Do not leave the default password unchanged.

---

## Create a Named Administrator Account

Navigate to:

```text
Users
→ Users
→ Create user
```

Example:

```text
Username: joco.admin
Name: József
Surname: Takács
Role: Super admin role
User group: Zabbix administrators
```

Set a strong password.

After verifying the new account, the default `Admin` account should be disabled or protected according to organizational policy.

---

## Configure HTTPS

The Zabbix frontend should use HTTPS.

Possible certificate sources include:

* internal certification authority;
* public certification authority;
* Let's Encrypt;
* reverse proxy certificate management.

Install Certbot for Nginx:

```bash
apt install certbot python3-certbot-nginx -y
```

Request a certificate:

```bash
certbot --nginx -d zabbix.example.com
```

Certbot can automatically:

* request the certificate;
* update Nginx;
* configure HTTPS;
* configure HTTP-to-HTTPS redirection.

Test renewal:

```bash
certbot renew --dry-run
```

Check the renewal timer:

```bash
systemctl status certbot.timer
```

Public certificates require:

* public DNS resolution;
* domain ownership;
* reachable validation endpoint;
* suitable firewall rules.

---

## Example Manual HTTPS Configuration

Example Nginx configuration:

```nginx
server {
    listen 80;
    server_name zabbix.example.com;

    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name zabbix.example.com;

    ssl_certificate /etc/letsencrypt/live/zabbix.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/zabbix.example.com/privkey.pem;

    root /usr/share/zabbix;
    index index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ [^/]\.php(/|$) {
        fastcgi_pass unix:/run/php/zabbix.sock;
        include fastcgi_params;

        fastcgi_param SCRIPT_FILENAME \
            /usr/share/zabbix$fastcgi_script_name;
    }
}
```

Test and reload Nginx:

```bash
nginx -t
systemctl reload nginx
```

---

## Configure Zabbix Agent 2 on the Server

Edit the Agent 2 configuration:

```bash
nano /etc/zabbix/zabbix_agent2.conf
```

Configure the server addresses:

```text
Server=127.0.0.1,192.168.10.20
ServerActive=127.0.0.1
Hostname=ZBX-SRV-01
```

Important:

```text
Hostname=
```

must match the host name created in the Zabbix frontend when active checks are used.

Restart the agent:

```bash
systemctl restart zabbix-agent2
```

Enable it:

```bash
systemctl enable zabbix-agent2
```

Check status:

```bash
systemctl status zabbix-agent2
```

---

## Test the Local Agent

Install `zabbix-get`:

```bash
apt install zabbix-get -y
```

Test agent availability:

```bash
zabbix_get -s 127.0.0.1 -k agent.ping
```

Expected output:

```text
1
```

Test hostname:

```bash
zabbix_get -s 127.0.0.1 -k system.hostname
```

Test system uptime:

```bash
zabbix_get -s 127.0.0.1 -k system.uptime
```

Test CPU count:

```bash
zabbix_get -s 127.0.0.1 -k system.cpu.num
```

Test memory:

```bash
zabbix_get -s 127.0.0.1 -k vm.memory.size[total]
```

---

## Add the Zabbix Server as a Monitored Host

In the frontend, navigate to:

```text
Data collection
→ Hosts
→ Create host
```

Configure:

```text
Host name: ZBX-SRV-01
Visible name: Zabbix Server
Host groups:
- Zabbix servers
- Linux servers
```

Add an agent interface:

```text
IP address: 127.0.0.1
Port: 10050
```

Assign templates:

```text
Linux by Zabbix agent
Zabbix server health
```

Template names may differ between versions.

Save the host.

---

## Verify Data Collection

Navigate to:

```text
Monitoring
→ Latest data
```

Filter by:

```text
Host: ZBX-SRV-01
```

Verify that values are received for:

* CPU utilization;
* memory usage;
* disk usage;
* network traffic;
* agent availability;
* Zabbix server processes;
* cache utilization;
* queue metrics;
* values processed per second.

If values are missing, verify:

* host name;
* agent interface;
* template assignment;
* firewall settings;
* agent configuration;
* server log;
* agent log.

---

## Configure Database Backup

A monitoring platform must have a tested database backup.

Create a backup directory:

```bash
mkdir -p /var/backups/zabbix
chown postgres:postgres /var/backups/zabbix
chmod 700 /var/backups/zabbix
```

Create a manual backup:

```bash
sudo -u postgres pg_dump \
    -Fc \
    -d zabbix \
    -f /var/backups/zabbix/zabbix_$(date +%F_%H-%M).dump
```

List backup files:

```bash
ls -lh /var/backups/zabbix
```

Verify the backup:

```bash
sudo -u postgres pg_restore \
    --list \
    /var/backups/zabbix/zabbix_YYYY-MM-DD_HH-MM.dump
```

A backup that has never been tested is not a verified backup.

---

## Automated Database Backup Script

Create a backup script:

```bash
nano /usr/local/sbin/zabbix-db-backup.sh
```

Example:

```bash
#!/bin/bash

set -euo pipefail

BACKUP_DIR="/var/backups/zabbix"
DATABASE="zabbix"
RETENTION_DAYS="14"
TIMESTAMP="$(date +%F_%H-%M-%S)"
BACKUP_FILE="${BACKUP_DIR}/${DATABASE}_${TIMESTAMP}.dump"

mkdir -p "${BACKUP_DIR}"

sudo -u postgres pg_dump \
    -Fc \
    -d "${DATABASE}" \
    -f "${BACKUP_FILE}"

find "${BACKUP_DIR}" \
    -type f \
    -name "${DATABASE}_*.dump" \
    -mtime +"${RETENTION_DAYS}" \
    -delete

echo "Backup completed: ${BACKUP_FILE}"
```

Make the script executable:

```bash
chmod 750 /usr/local/sbin/zabbix-db-backup.sh
```

Test it:

```bash
/usr/local/sbin/zabbix-db-backup.sh
```

---

## Schedule the Backup

Edit root crontab:

```bash
crontab -e
```

Example daily backup at 02:00:

```cron
0 2 * * * /usr/local/sbin/zabbix-db-backup.sh >> /var/log/zabbix-db-backup.log 2>&1
```

Verify scheduled jobs:

```bash
crontab -l
```

Database backups should also be copied to:

* another server;
* backup storage;
* immutable storage;
* off-site storage.

Keeping backups only on the Zabbix server does not protect against complete server failure.

---

## Back Up Configuration Files

Important configuration paths include:

```text
/etc/zabbix/
/etc/nginx/
/etc/php/
/etc/postgresql/
/etc/letsencrypt/
/usr/lib/zabbix/
/usr/local/share/zabbix/
```

Example archive:

```bash
tar -czf /var/backups/zabbix/zabbix-config_$(date +%F).tar.gz \
    /etc/zabbix \
    /etc/nginx \
    /etc/postgresql
```

Certificate directories may require separate handling due to sensitive private keys.

---

## Test Database Restoration

Create a temporary restore database:

```bash
sudo -u postgres createdb zabbix_restore_test
```

Restore the backup:

```bash
sudo -u postgres pg_restore \
    -d zabbix_restore_test \
    /var/backups/zabbix/zabbix_YYYY-MM-DD_HH-MM.dump
```

Verify the restored tables:

```bash
sudo -u postgres psql \
    -d zabbix_restore_test \
    -c "SELECT count(*) FROM hosts;"
```

Delete the test database:

```bash
sudo -u postgres dropdb zabbix_restore_test
```

Restore tests should be performed regularly.

---

## Configure Log Rotation

Zabbix packages normally install logrotate configuration.

Verify:

```bash
cat /etc/logrotate.d/zabbix-server
cat /etc/logrotate.d/zabbix-agent2
```

Test logrotate configuration:

```bash
logrotate -d /etc/logrotate.conf
```

Check log file sizes:

```bash
du -sh /var/log/zabbix
du -sh /var/log/nginx
du -sh /var/log/postgresql
```

Uncontrolled log growth can exhaust the root filesystem.

---

## Secure SSH Access

Recommended SSH controls include:

* key-based authentication;
* disabled direct root login;
* restricted source networks;
* limited administrative users;
* logging and monitoring;
* disabled password authentication where appropriate.

Edit SSH configuration:

```bash
nano /etc/ssh/sshd_config
```

Example settings:

```text
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AllowUsers monitoringadmin
```

Validate the configuration:

```bash
sshd -t
```

Restart SSH:

```bash
systemctl restart ssh
```

Do not disable password authentication before confirming that key-based login works.

---

## Create an Administrative User

Create a named administrative account:

```bash
adduser monitoringadmin
```

Grant sudo access:

```bash
usermod -aG sudo monitoringadmin
```

Verify:

```bash
id monitoringadmin
```

Configure SSH keys:

```bash
mkdir -p /home/monitoringadmin/.ssh
nano /home/monitoringadmin/.ssh/authorized_keys
```

Set permissions:

```bash
chown -R monitoringadmin:monitoringadmin \
    /home/monitoringadmin/.ssh

chmod 700 /home/monitoringadmin/.ssh
chmod 600 /home/monitoringadmin/.ssh/authorized_keys
```

---

## Disable Unnecessary Services

List enabled services:

```bash
systemctl list-unit-files --state=enabled
```

List listening ports:

```bash
ss -tulpn
```

Disable unused services only after confirming their purpose.

Example:

```bash
systemctl disable --now <service-name>
```

Do not disable:

* networking;
* SSH;
* PostgreSQL;
* Nginx;
* PHP-FPM;
* Zabbix Server;
* Zabbix Agent 2;
* time synchronization.

---

## Configure Automatic Security Updates

Install unattended upgrades:

```bash
apt install unattended-upgrades apt-listchanges -y
```

Configure:

```bash
dpkg-reconfigure unattended-upgrades
```

Review:

```text
/etc/apt/apt.conf.d/50unattended-upgrades
/etc/apt/apt.conf.d/20auto-upgrades
```

Automatic upgrades should be aligned with the organization's change-management policy.

Critical production systems may require:

* maintenance windows;
* pre-production testing;
* snapshot procedures;
* rollback planning.

---

## Verify Database Health

Check PostgreSQL service:

```bash
systemctl status postgresql
```

Check database size:

```bash
sudo -u postgres psql -d zabbix -c \
"SELECT pg_size_pretty(pg_database_size('zabbix'));"
```

Check active sessions:

```bash
sudo -u postgres psql -d zabbix -c \
"SELECT count(*) FROM pg_stat_activity;"
```

Check long-running queries:

```bash
sudo -u postgres psql -d zabbix -c "
SELECT
    pid,
    now() - query_start AS duration,
    state,
    query
FROM pg_stat_activity
WHERE state <> 'idle'
ORDER BY duration DESC;
"
```

---

## Verify Zabbix Internal Health

In the frontend, review:

```text
Monitoring
→ Hosts
→ Zabbix Server
```

Important internal metrics include:

* Zabbix server running;
* values processed per second;
* queue size;
* unsupported items;
* poller utilization;
* preprocessing utilization;
* history syncer utilization;
* cache utilization;
* database availability;
* process busy percentages.

Persistently high process utilization may require:

* item interval review;
* process tuning;
* database optimization;
* proxy deployment;
* template cleanup.

---

## Installation Validation Commands

Use the following commands for a final system check.

### Operating system

```bash
hostnamectl
timedatectl
uptime
df -h
free -h
```

### Network

```bash
ip address show
ip route show
ss -tulpn
ping -c 4 192.168.10.1
```

### Services

```bash
systemctl is-active postgresql
systemctl is-active nginx
systemctl is-active php8.2-fpm
systemctl is-active zabbix-server
systemctl is-active zabbix-agent2
```

### Database

```bash
sudo -u postgres psql -d zabbix -c "SELECT version();"
```

### Zabbix Agent

```bash
zabbix_get -s 127.0.0.1 -k agent.ping
```

### Web frontend

```bash
curl -I http://127.0.0.1
```

After HTTPS configuration:

```bash
curl -I https://zabbix.example.com
```

---

## Installation Checklist

### Operating system

* [ ] Debian installed
* [ ] System packages updated
* [ ] Static IP configured
* [ ] Hostname configured
* [ ] DNS resolution working
* [ ] Correct timezone configured
* [ ] Time synchronization active

### Database

* [ ] PostgreSQL installed
* [ ] Zabbix database created
* [ ] Zabbix database user created
* [ ] Strong database password configured
* [ ] Initial schema imported
* [ ] Database connection tested
* [ ] Database backup created
* [ ] Restore test completed

### Zabbix components

* [ ] Zabbix repository installed
* [ ] Zabbix Server installed
* [ ] Zabbix Frontend installed
* [ ] Zabbix Agent 2 installed
* [ ] Zabbix database parameters configured
* [ ] Zabbix Server running
* [ ] Zabbix Agent 2 running
* [ ] Local agent test successful

### Web frontend

* [ ] Nginx installed
* [ ] PHP-FPM installed
* [ ] PHP timezone configured
* [ ] Nginx configuration validated
* [ ] Frontend wizard completed
* [ ] Default administrator password changed
* [ ] Named administrator created
* [ ] HTTPS enabled

### Security

* [ ] Firewall enabled
* [ ] SSH restricted
* [ ] Root SSH login disabled
* [ ] Database not exposed publicly
* [ ] Configuration file permissions reviewed
* [ ] Automatic security updates configured
* [ ] Unnecessary services disabled

### Monitoring readiness

* [ ] Zabbix server added as monitored host
* [ ] Linux template assigned
* [ ] Zabbix server health template assigned
* [ ] Latest data visible
* [ ] Internal metrics collected
* [ ] Notification configuration planned

---

## Common Installation Problems

### Zabbix Server Does Not Start

Check:

```bash
systemctl status zabbix-server
journalctl -u zabbix-server -n 100 --no-pager
tail -n 100 /var/log/zabbix/zabbix_server.log
```

Common causes:

* incorrect database password;
* database unavailable;
* schema not imported;
* unsupported configuration parameter;
* permission error;
* port conflict.

---

### Database Connection Failed

Test manually:

```bash
psql -h 127.0.0.1 -U zabbix -d zabbix
```

Verify:

```text
DBHost=
DBName=
DBUser=
DBPassword=
```

Check PostgreSQL authentication:

```bash
cat /etc/postgresql/*/main/pg_hba.conf
```

---

### Frontend Displays a Blank Page

Check:

```bash
tail -n 100 /var/log/nginx/error.log
journalctl -u php8.2-fpm -n 100 --no-pager
```

Common causes:

* PHP-FPM not running;
* wrong PHP socket;
* missing PHP extension;
* incorrect permissions;
* invalid Nginx configuration.

---

### Zabbix Agent Is Unreachable

Check:

```bash
systemctl status zabbix-agent2
ss -lntp | grep 10050
```

Test locally:

```bash
zabbix_get -s 127.0.0.1 -k agent.ping
```

Verify:

```text
Server=
ServerActive=
Hostname=
```

Check the firewall:

```bash
ufw status verbose
```

---

### Hostname Mismatch

Active checks require an exact hostname match.

Agent configuration:

```text
Hostname=ZBX-SRV-01
```

Frontend host name:

```text
ZBX-SRV-01
```

These values must match exactly.

---

### Nginx Configuration Test Fails

Run:

```bash
nginx -t
```

Review the reported file and line number.

Common causes:

* duplicate `listen` directive;
* missing semicolon;
* invalid certificate path;
* duplicate server block;
* incorrect PHP socket.

---

## Production Recommendations

For production use:

* place the server on reliable SSD or NVMe storage;
* use a static IP address;
* use proper DNS records;
* enable HTTPS;
* restrict SSH and frontend access;
* use a dedicated database password;
* back up the database daily;
* copy backups off the server;
* monitor database growth;
* monitor Zabbix internal processes;
* document all configuration changes;
* test restoration procedures;
* use a Zabbix proxy for remote sites;
* avoid public exposure of agents and SNMP services.

---

## Summary

A successful Zabbix installation requires more than installing software packages.

The deployment must include:

* a supported operating system;
* reliable database storage;
* correct network configuration;
* secure web access;
* protected credentials;
* tested backups;
* self-monitoring;
* documented recovery procedures.

After installation, the next step is to create, organize and manage monitored hosts.

[Next: Host Management →](04-host-management.md)

