# ansible-swarm-mempool
Ansible repo for running Mempool and monitoring stack on Docker Swarm

## Overview
Three hosts are configured to run Docker in swarm mode.

The swarm will run an instance of Mempool along with an Electrum server and Traefik to provide https access. Host and container level monitoring is performed using Prometheus, with metrics viewable in Grafana. Alerts are sent to Discord. A Certificate Authority is set up on the Ansible control host to issue certificates for authentication between services.

## Docker modifications
The Docker daemon is modified to expose metrics for Prometheus. Also, TLS access to the socket is enabled with client and server authentication.

## Swarm services
### Traefik
An instance of https://github.com/traefik/traefik.

This provides https access to resources that should be accessible to humans. Trusted certs for a Cloudflare managed domain are issued via ACME.

#### Required variables
| Variable | Description |
|---|:---:|
|traefik_domain|Cloudflare managed domain name. Used by other services|
|traefik_acme_email|Email address|
|traefik_cf_dns_api_token|Cloudflare API token with Zone Read and DNS Edit rights|

### Mempool
A private version of [mempool.space](https://mempool.space).

Deployed as a stack using a modified version of a Docker Compose file found at
https://github.com/mempool/mempool.

Requires access to a Bitcoin node with RPC access enabled.

#### Required variables
| Variable | Description |
|---|:---:|
|mempool_rpc_host|IP or hostname of Bitcoin node|
|mempool_rpc_user|Bitcoin node RPC username|
|mempool_rpc_pass|Bitcoin node RPC password|
|mempool_mysql_user|Arbitrary username to connect to MariaDB instance|
|mempool_mysql_pass|Arbitrary password to connect to MariaDB instance|
|mempool_mysql_root_pass|Arbitrary root password for MariaDB instance|

### Electrumx
A Docker version of https://github.com/spesmilo/electrumx.

Requires access to a Bitcoin full node with txindex=1 and RPC access enabled
to connect to and sync.

#### Required variables
| Variable | Description |
|---|:---:|
|electrumx_daemon_url|Connection string to Bitcoin node ie http://user:pass@host:port| 

### Prometheus
Prometheus scrapes cAdvisor instances running as a global service in the swarm to obtain container level metrics and node_exporter instances running on each host as a systemd service for host level metrics. 'dockerswarm_sd_config' is used to discover the Swarm nodes, cAdvisor and node_exporter instances, and alertmanager.

### Alertmanager / Alertmanager-discord
Alertmanager receives alerts from Prometheus and forwards them to the alertmanager-discord service as a webhook so that notifications are sent to Discord.
#### Required variables
| Variable | Description |
|---|:---:|
|discord_webhook|URL of Discord webhook|

### Grafana
Grafana uses Prometheus as a datasource to graph collected metrics.

## Swarm storage
Since Docker Swarm does not include a way to persist storage between nodes,
NFS shares are mounted by services that need to save data persistently.

#### Required variables
| Variable | Description |
|---|:---:|
|nfs_mount_options|NFS mount info ie "addr=###.###.###.###,rw,nfsvers=4,async"|
