## Ansible Infrastructure Automation

This repository contains a fully automated deployment of a multi-server virtual infrastructure using Ansible.
The project demonstrates Infrastructure as Code (IaC) principles and includes automated configuration of:

* DNS (Bind)
* DNS Clients
* NTP Server & Clients
* Web Server (Apache + Virtual Hosts)
* MariaDB Database
* Nagios Core Monitoring + NRPE Agents
* NFS Server
* AutoFS Clients
* Docker Manager + Workers (optional)

All components are deployed through modular Ansible roles for maximum reusability and clarity.


## Topology

All hosts ip range `192.168.3.0/24`:

| Role           | IP            | FQDN                                  | Alias (CNAME)         |
|----------------|---------------|---------------------------------------|-----------------------|
| Ansible master | 192.168.3.200 | `prdx-ansible101.ziyotek.local`       | `ansible`             |
| DNS            | 192.168.3.201 | `prdx-nsprimary101.ziyotek.local`     | `dns`                 |
| NTP            | 192.168.3.202 | `prdx-ntp101.ziyotek.local`           | `ntp`                 |
| Database       | 192.168.3.203 | `prdx-db101.ziyotek.local`            | `db`                  |
| Web Server     | 192.168.3.204 | `prdx-webserver101.ziyotek.local`     | `webserver`           |
| Load balancer  | 192.168.3.205 | `prdx-haproxy101.ziyotek.local`       | `loadbalancer`        |
| Nagios         | 192.168.3.206 | `prdx-nagios101.ziyotek.local`        | `nagios`              |
| FTP            | 192.168.3.207 | `prdx-ftp101.ziyotek.local`           | `ftp`                 |
| NFS            | 192.168.3.208 | `prdx-nfs101.ziyotek.local`           | `nfs`                 |
| Docker manager | 192.168.3.209 | `prdx-dprimary101.ziyotek.local`      | `docker`              |
| Docker worker1 | 192.168.3.210 | `prdx-dworker101.ziyotek.local`       | `dockw1`              |
| Docker worker2 | 192.168.3.211 | `prdx-dworker102.ziyotek.local`       | `dockw2`              |

All those hosts described in `hosts.ini` like: `ansible`, `dns`, `ntp`, `database`, `web`, `loadbalancer`, `nagios`, `ftp`, `nfs`, `docker_manager`, `docker_workers`, `clients`.


## Repository Structure
.
├── ansible.cfg                  # Main Ansible settings: inventory, SSH options, privilege escalation
├── hosts.ini                    # Inventory file with host groups and variables
├── README.md                    # Project documentation (this file)
|
├── roles/                       # All automation roles
│   ├── dns/                     # Bind DNS Server role
│   │   ├── tasks/
│   │   │   └── main.yml         # Install & configure Bind, zones, firewalld
│   │   ├── templates/
│   │   │   ├── named.conf.j2
│   │   │   ├── forward-zone.db.j2
│   │   │   └── reverse-zone.db.j2
│   │   └── handlers/
│   │       └── main.yml         # Restart named
│   │
│   ├── dns_clients/             # DNS Clients role
│   │   └── tasks/main.yml       # Configure resolv.conf or nmcli
│   │
│   ├── ntp/                     # NTP Server role
│   │   ├── tasks/main.yml
│   │   └── templates/chrony.conf.j2
│   │
│   ├── ntp_client/              # NTP Clients role
│   │   ├── tasks/main.yml
│   │   └── templates/chrony.client.conf.j2
│   │
│   ├── web/                     # Apache Web Server + Virtual Hosts
│   │   ├── tasks/main.yml
│   │   ├── templates/
│   │   │   ├── vhost.conf.j2    # Apache vhost template
│   │   │   └── index.html.j2    # Per-site webpage
│   │   └── handlers/main.yml    # Restart httpd
│   │
│   ├── mariadb/                 # MariaDB Server role
│   │   ├── tasks/main.yml
│   │   ├── templates/my.cnf.j2
│   │   └── handlers/main.yml
│   │
│   ├── nagios/                  # Nagios Core Server role
│   │   ├── tasks/main.yml       # Install Nagios from source + configs
│   │   ├── templates/
│   │   │   ├── hosts.cfg.j2     # All monitored hosts list
│   │   │   └── services.cfg.j2  # NRPE service checks
│   │   └── handlers/main.yml    # Restart Nagios + Apache
│   │
│   ├── nrpe/                    # NRPE client role
│   │   ├── tasks/main.yml       # Install nrpe + plugins + ACL for Nagios server
│   │   └── templates/nrpe.cfg.j2
│   │
│   ├── nfs_server/              # NFS Export Server role
│   │   ├── tasks/main.yml
│   │   ├── templates/exports.j2
│   │   └── handlers/main.yml
│   │
│   ├── autofs_client/           # AutoFS mount clients role
│   │   ├── tasks/main.yml
│   │   ├── templates/
│   │   │   ├── auto.master.j2   # AutoFS main config
│   │   │   └── auto.nfs.j2      # Map for /mnt/nfs/data mount
│   │   └── handlers/main.yml
│   │
│   └── docker/ (optional)       # Docker Swarm automation
|
├── site.yml                     # Main entry point: includes all roles in correct order



## Explanation of Each Automated Component

DNS Server (Bind9)
* Installs Bind
* Creates forward & reverse zones from templates
* Manages named.conf
* Allows queries only from local subnet
* Opens DNS port in firewalld


DNS Clients
All clients automatically get:

  nameserver 192.168.3.201
  search ziyotek.local


NTP Server & Clients (Chrony)

Server (192.168.3.202):
* Uses external pool servers
* Provides time for entire network
* Firewalld allows NTP service

Clients:
* Sync with internal server
* Autostart & enable chronyd

Web Server (Apache + Virtual Hosts)
Three websites are deployed automatically:

  vhost1.ziyotek.local
  vhost2.ziyotek.local
  vhost3.ziyotek.local


Each site gets:
* Its own document root under /var/www/vhosts/
* A custom index page
* A dedicated Apache configuration in /etc/httpd/conf.d/

MariaDB Database Server

Role performs:
* Installation of MariaDB
* Root password setup
* Service enablement
* Optional DB creation

Nagios Core Monitoring

The Nagios server deployment includes:
* Compilation from source
* Apache integration
* Web UI at:

  http://<nagios_ip>/nagios


Credentials:

  nagiosadmin / Nagios123

It also:
* Installs check_nrpe plugin
* Configures commands.cfg
* Uses /usr/local/nagios/etc/servers/   for modular configs

NRPE Agents (Clients)

Installed on all clients except:
* Nagios server itself
* Ansible controller

Each client returns:
* CPU load
* Disk usage
* Swap
* Processes
* Ping

NFS Server

NFS exports:

  /srv/nfs/data 192.168.3.0/24(rw,sync,no_root_squash,no_subtree_check)


Services enabled:
* nfs-server
* mountd
* rpc-bind
  Ports opened via firewalld.

AutoFS Clients

Automatically mount:

  /mnt/nfs/data → 192.168.3.208:/srv/nfs/data

Using:
* /etc/auto.master
* /etc/auto.nfs
Mounting is on-demand.

How to Deploy
Run the full infrastructure:

  ansible-playbook site.yml


Run only specific roles:

  ansible-playbook site.yml --limit dns
  ansible-playbook site.yml --limit web
  ansible-playbook site.yml --limit nagios
  ansible-playbook site.yml --limit "clients:!nfs"

jhfgjhfg
lkhjlkh


Verification Checklist
DNS:
  dig vhost1.ziyotek.local
  dig -x 192.168.3.204

NTP:
  chronyc sources

Web:
  curl http://vhost1.ziyotek.local

NFS:
  ls /mnt/nfs/data

AutoFS:
  cd /mnt/nfs/data

Nagios:
Open browser:

  http://192.168.3.206/nagios


## Conclusion

This project demonstrates a complete automated infrastructure deployment using Ansible roles.
The repository follows best practices for maintainability and scalability.