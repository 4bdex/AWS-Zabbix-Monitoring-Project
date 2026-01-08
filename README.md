# AWS Zabbix Monitoring Project

Centralized Zabbix monitoring on AWS for a hybrid fleet (Linux and Windows) using a containerized stack.

## Goals
- Deploy a reproducible Zabbix stack with Docker Compose on AWS.
- Collect CPU, RAM, and network metrics from heterogeneous hosts.
- Provide a simple path to onboard Linux and Windows clients securely.

## Architecture

<div align="center">

![Architecture](images/archetecture.png)

</div>

- **VPC:** Project-Zabbix-VPC, CIDR 10.0.0.0/16, public subnet 10.0.1.0/24, internet gateway for UI access.
- **Security groups:**
	- SG-Zabbix-Server: allow 80/443 (UI) and 10051 (agent data in).
	- SG-Clients: allow 10050 (server poll), plus SSH/RDP for admin.
- **Instances:**
	- Zabbix server: Ubuntu 22.04 LTS (t3.large) running Docker stack.
	- Linux client: Ubuntu 22.04 LTS (t3.medium).
	- Windows client: Windows Server 2022 (t3.large).

## Stack Components
- `zabbix-server-mysql`: Zabbix server core.
- `zabbix-web-nginx-mysql`: Web UI fronted by Nginx.
- `mysql-server`: Metrics datastore.

Ports in use: 80/443 (UI), 10051 (server), 10050 (agents).

## Prerequisites
- AWS account with the VPC, subnet, and security groups above.
- Docker and Docker Compose installed on the Zabbix server host.
- Security group rules open from clients to the server on port 10051/10050 as needed.

## Quick Start
1) Clone this repo on the Zabbix server host.
2) From the repo root, start the stack:
	 ```sh
	 docker compose up -d
	 ```
3) Open the UI at `http://<Zabbix-Server-Public-IP>` and complete the initial setup wizard.

## Configure Zabbix Agents
- **Linux (example Ubuntu):**
	1) Install agent: `sudo apt install zabbix-agent` (or use the official repo).
	2) Edit `/etc/zabbix/zabbix_agentd.conf`: set `Server=<server-private-ip>` and `ServerActive=<server-private-ip>`.
	3) Restart: `sudo systemctl restart zabbix-agent` and ensure port 10050 is allowed inbound.

- **Windows:**
	1) Install the official MSI.
	2) Set `Server` and `ServerActive` to the server private IP during install.
	3) Allow port 10050 inbound in Windows Firewall.

## Add Hosts in Zabbix UI
1) In the UI, go to Configuration → Hosts → Create host.
2) Set the host name and visible name; add to a group.
3) Under Interfaces, add an Agent interface pointing to the client IP.
4) Link templates (e.g., OS Linux or Windows) to collect CPU, RAM, and network metrics.
5) Save and confirm availability status turns green.

## Monitoring Outcomes
- Metrics appear every minute: CPU usage, memory consumption, and network throughput.
- Availability icon confirms agent reachability; graphs can be viewed under Monitoring → Hosts.

## Maintenance and Notes
- To update the stack: `docker compose pull && docker compose up -d`.
- Check container health: `docker compose ps`.
- Logs: `docker compose logs -f zabbix-server-mysql` (or other services).