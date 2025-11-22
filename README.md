# Ansible infrastucture of 12 hosts


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


## Structure of repository

.
├── ansible.cfg        # Main settings Ansible (inventory, remote_users etc)
├── hosts.ini          # Inventory file with all hosts and groups
├── README.md          # Describe of Project (this file)
├── playbooks/
│   ├── 1_dns.yml          # Settings DNS server (bind)
│   ├── 2_dns_clients.yml  # Settings DNS clients (nmcli, resolv)
│   ├── 3_ntp.yml          # Settings NTP server and clients (chrony)
│   └── templates/
│       ├── ziyotek.local.db.j2                # forward-zone
│       ├── reverse.3.168.192.in-addr.arpa.j2  # reverse-zone
│       ├── chrony.server.conf.j2              # config chrony for NTP-server
│       └── chrony.client.conf.j2              # config chrony for NTP-clients
└── Archive/
    └── ... # Reserv copy olds playbook's (.bak)
